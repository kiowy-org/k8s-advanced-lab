# Mise à jour de Kubernetes avec kubeadm

Dans ce TP, nous allons utliser l'outil officiel afin de mettre à jour Kubernetes.

## 1. Infrastructure disponible

Nous allons utiliser le cluster réalisé au TP1

| Host                | Public hostname                     |
| ------------------- | ----------------------------------- |
| controller-0-prenom | controller-0-prenom.forma.kiowy.net |
| worker-0-prenom     | worker-0-prenom.forma.kiowy.net     |
| worker-1-prenom     | worker-1-prenom.forma.kiowy.net     |



## 2. Mise à jour du control plane

Nous allons commencer par mettre à jour les composants du control plane (master) :

**L'objectif et de passer en 1.33 !**

Sur le noeud master :

```shell
{
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt-cache madison kubeadm
}
```

Déterminez la version du package à installer, puis mettez à jour kubeadm.

```shell
{
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.*' && \
sudo apt-mark hold kubeadm
}
```

Vérifiez la version de kubeadm, puis génerez le plan de mise à jour.


```shell
{
kubeadm version
sudo kubeadm upgrade plan
}
```

Démarrez le processus de mise à jour mais... n'oubliez pas de retirer les pods du noeud master !

```shell
{
kubectl drain <master> --ignore-daemonsets
# Indiquez la version choisie au format #v1.33.xx
sudo kubeadm upgrade apply <version>
}
```

Maintenant, il ne reste plus qu'à mettre à jour le kubelet et kubectl.


```shell
{
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.*' kubectl='1.33.*' && \
sudo apt-mark hold kubelet kubectl
}
```

Enfin, redémarrez le kubelet et vérifiez que votre mise à jour a fonctionnée.

```shell
{
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <master>
}
```


## 3. Mise à jour des noeuds workers

Une fois le master à jour, vous pouvez continuer le processus sur les workers.

Comme pour le master, mettez à jour kubeadm vers la 1.33

```shell
{
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.*' && \
sudo apt-mark hold kubeadm
}
```

Drainez le node avant la mise à jour.

```shell
{
kubectl drain <node> --ignore-daemonsets
}
```

Puis lancez le processus de mise à jour.

```shell
{
sudo kubeadm upgrade node
}
```

Installez ensuite la nouvelle version du kubelet et de kubectl.

```shell
{
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.*' kubectl='1.33.*' && \
sudo apt-mark hold kubelet kubectl
}
```

Enfin, redémarrez le kubelet, puis rendez le noeud à nouveau schedulable. Vérifiez que votre cluster fonctionne bien.

```shell
{
sudo systemctl daemon-reload
sudo systemctl restart kubelet
}
```

```shell
{
kubectl uncordon <node>
}
```
