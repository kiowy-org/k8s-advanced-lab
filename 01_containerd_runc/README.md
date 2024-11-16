# Conteneurisation avec runC et containerd

Dans ce TP, nous allons manipuler des conteneurs à l'aide du runtime bas niveau runC, ainsi qu'avec le runtime haut niveau containerd. 

## 1. Installation

Connectez vous via ssh à la VM de l'exercice : `ssh -i id_forma etudiant@runc-<prenom>.forma.kiowy.net`

Une fois connecté sur votre machine (vous devez voir le prompt apparaitre), exécutez les commandes suivantes, afin d'installer 3 composants : `runc`, `containerd` et `crictl`.

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.31.1/crictl-v1.31.1-linux-amd64.tar.gz \
  https://github.com/opencontainers/runc/releases/download/v1.2.2/runc.amd64 \
  https://github.com/containerd/containerd/releases/download/v2.0.0/containerd-2.0.0-linux-amd64.tar.gz \
  https://github.com/containernetworking/plugins/releases/download/v1.6.0/cni-plugins-linux-amd64-v1.6.0.tgz
```

```bash
{
  mkdir containerd
  sudo mkdir -p /opt/cni/bin
  tar -xvf crictl-v1.31.1-linux-amd64.tar.gz
  tar -xvf containerd-2.0.0-linux-amd64.tar.gz -C containerd
  sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.0.tgz
  sudo mv runc.amd64 runc
  chmod +x crictl runc 
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}
```

Vérifiez l'installation en exécutant `runc` et `crictl`.

## 2. Création de conteneurs avec runC

Nous allons créer un conteneur basiquement, à l'aide de runC. Pour démarrer le conteneur, runC a besoin de deux informations : le rootfs (arborescence du conteneur à créer) et un fichier de configuration (expliquant la commande à exécuter au démarrage par exemple).

Un archive tar de l'image busybox est fournie dans ce TP, vous devez copier l'archive sur la machine de TP :
```bash
wget -q --show-progress --https-only --timestamping https://github.com/kiowy-org/k8s-advanced-lab/raw/refs/heads/master/01_containerd_runc/busyboximg.tar
```

Préparez ensuite les dossier pour construire notre conteneur.
```bash
{
  mkdir monconteneur
  mkdir monconteneur/rootfs
  tar -C monconteneur/rootfs -xvf busyboximg.tar
  cd ./monconteneur
}
```

Explorez le contenu de `rootfs`, que contient-il ?

Pour que notre conteneur démarre, il manque son fichier de configuration. runC peut en générer un grâce à la commande 
```bash
runc spec
```

Une fois le fichier `config.json` généré, démarrez votre conteneur.
```bash
sudo runc run monsuperconteneur
```
Si vous voyez un prompt sh, félicitation, vous venez de démarrer votre premier conteneur directement avec un runtime bas niveau ! Pour sortir tapez `exit`.

## 3. Explorons le fonctionnement du conteneur

Pour l'instant, nous avons démarré un conteneur interactif (il termine dès que nous le quittons). Nous allons démarrer un conteneur non interactif, afin de bien voir son fonctionnement sur le système hôte.

Éditez le fichier `config.json` afin que le conteneur soit non interactif (`terminal: false`), et n'oubliez pas de changer la commande qui sera exécutée au démarrage, par une commande longue (ex. `sleep 180`).

Créez ensuite le conteneur sans le démarrer :
```bash
  sudo runc create myconteneurid
```

Vous pouvez lister les conteneurs via la commande `sudo runc list`. Vérifiez que votre conteneur est créé mais n'est pas encore démarré. 

Puis démarrez le conteneur avec `sudo runc start myconteneurid`. Vérifiez son état, puis exécutez la commande `ps -aux | grep sleep`. Que constatez vous ?

Réexecutez la commande `runc list` et `ps` quand votre commande `sleep` est terminée.

Enfin, vous pouvez supprimer votre conteneur :
```shell
sudo runc delete myconteneurid
```

## 4. Utilisation de containerd et crictl

crictl est un outil ligne de commande capable de fonctionner avec tout les runtimes de conteneurs respectant la Container Runtime Interface (CRI). Nous allons donc pouvoir l'utiliser pour manipuler containerd.

containerd utilise un service afin de fonctionner, nous allons le configurer puis le démarrer.

```bash
sudo mkdir -p /etc/containerd/
```

```shell
containerd config default | sudo tee /etc/containerd/config.toml
```
Enfin, il faut créer un fichier `containerd.service` afin que systemd puisse gérer notre service.
```shell
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
Enfin, nous pouvons démarrer le daemon containerd.
```shell
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd
  sudo systemctl start containerd
}
```

## 5. Démarrons notre conteneur busybox via crictl
Contrairement à runC qui ne comprend que le concept de conteneur, containerd peut pull des images et les utiliser directement. Nous pouvons donc pull une image busybox :
```shell
sudo crictl pull busybox
```

Vous pouvez lister les images via `crictl images`.

Pour exécuter notre conteneur, il faut prendre en compte que crictl est un outil respectant la CRI de Kubernetes. crictl peut donc démarrer un Pod.

Pour cela, nous devons d'abord donner une définition globale de notre Pod, appelée sandbox.

```shell
cat <<EOF | tee pod-config.json
{
    "metadata": {
        "name": "busybox-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}
EOF
```

Ensuite, nous pouvons ajouter la définition de notre conteneur busybox :
```shell
cat << EOF | tee container-config.json
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "log_path":"busybox.0.log",
  "linux": {
  }
}
EOF
```

Enfin nous pouvons créer le pod :

```shell
sudo crictl runp pod-config.json
```
Que constatez vous ?

### Configurer la Container Network Interface

```shell
sudo mkdir -p /etc/cni/net.d
cat <<EOF | sudo tee /etc/cni/net.d/10-mynet.conf 
{
	"cniVersion": "0.2.0",
	"name": "mynet",
	"type": "bridge",
	"bridge": "cni0",
	"isGateway": true,
	"ipMasq": true,
	"ipam": {
		"type": "host-local",
		"subnet": "10.22.0.0/16",
		"routes": [
			{ "dst": "0.0.0.0/0" }
		]
	}
}
EOF
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf 
{
	"cniVersion": "0.2.0",
	"name": "lo",
	"type": "loopback"
}
EOF
```
Maintenant, tentons à nouveau de démarrer notre Pod :
(N'hésitez pas à observer ce qui se passe entre chaque commandes grâce à `crictl pods` ou encore `crictl ps -a`)
```shell
sudo crictl runp pod-config.json
sudo crictl create <POD ID> container-config.json pod-config.json
sudo crictl start <CONTENEUR ID>
```

Félicitations, vous savez désormais ce qui se passe pour nos conteneurs avec Kubernetes !
