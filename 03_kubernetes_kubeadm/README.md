# Installation de Kubernetes avec kubeadm

Dans ce TP, nous allons utliser l'outil officiel afin d'installer Kubernetes. Je vous rassure, ce sera bien plus simple que le TP précédent.

## 1. Infrastructure disponible

Nous allons déployer Kubernetes avec 1 master et 2 workers, les composants du controle-plane seront installés dans des pods :

| Host                | Public hostname                     |
| ------------------- | ----------------------------------- |
| controller-0-prenom | controller-0-prenom.forma.kiowy.net |
| worker-0-prenom     | worker-0-prenom.forma.kiowy.net     |
| worker-1-prenom     | worker-1-prenom.forma.kiowy.net     |

## 2. Configuration réseau de iptables

Nous devons tout d'abord s'assurer de la configuration de iptables (afin de voir le traffic sur les interface bridge)

Exécutez les commandes suivantes sur l'ensemble de vos machines :

```shell
{
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
}
```

## 3. Installation du runtime de conteneur

Nous allons ensuite installer le runtime sur chacun des machines (master et workers) :

```shell
{
sudo apt-get update && sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
}
```

## 4. Désactivation de la mémoire swap

Kubernetes nécessite également que la mémoire swap soit désactivée pour fonctionner :

Exécutez ce code sur tous les noeuds :

```shell
{
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
}
```

## 5. Installation du kubelet, de kubeadm et de kubectl

Comme les composants du controle-plane seront installés dans des conteneurs, nous n'avons que kubelet à installer sur l'ensemble de nos machines.
Nous allons également installer kubeadm et kubectl afin de réaliser les commandes nécessaires depuis tous nos noeuds si besoin

```shell
{
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo mkdir /etc/apt/keyrings
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
}
```

## 6. Initialisation du master

Désormais, tous les éléments nécessaires sont installés. Nous pouvons donc passer à l'initialisation du controle-plane.

Exécutez la commande suivante **uniquement sur le master !!**

```shell
sudo kubeadm init --pod-network-cidr 192.168.0.0/16
```

Cette commande va se charger de l'ensemble des étapes d'initialisation des composants du controle-plane pour nous. 
Une fois terminé, nous pouvons récupérer l'accès administrateur au cluster, afin d'éxecuter `kubectl` pour parler à l'API.

```shell
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Enfin, vous pouvez désormais exécuter `kubectl version` et voir la version du cluster.

## 7. Ajouter le plugin Calico (CNI)

La dernière étape consiste à déployer un addon CNI pour assurer le fonctionnement du réseau des pods. 
Nous pouvons installer Calico avec la commande suivante :

```shell
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Vérifiez le bon démarrage des composants de calico via `kubectl get pods -n kube-system`

## 8. Joindre les nodes au cluster

Il ne reste plus qu'une chose à faire, joindre nos nodes au cluster ! 
Pour cela, il vous faut une commande avec des identifiants afin d'autoriser les nodes à parler à l'API.

Utilisez la commande suivante **sur le master !!** :
```shell
sudo kubeadm token create --print-join-command
```


Copiez la commande retournée, connectez-vous ensuite sur vos workers et éxecutez là  (sans oublier sudo) !

Voilà ! Il ne vous reste plus qu'à vous rendre de nouveaux sur le master, et exécuter `kubectl get nodes`. 
Félicitations, vous pouvez profiter de votre acomplissement.
