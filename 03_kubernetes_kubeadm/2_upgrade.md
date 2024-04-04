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

**L'objectif et de passer en 1.29.1 seulement !**

Sur le noeud master :

```shell
{
sudo apt update
sudo apt-cache madison kubeadm
}
```

Déterminez la version du package à installer, puis mettez à jour kubeadm.

```shell
{
# remplacez 1.29.x-* par la version choisie
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.29.x-*' && \
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
sudo kubeadm upgrade apply v1.29.x
}
```

Maintenant, il ne reste plus qu'à mettre à jour le kubelet et kubectl.


```shell
{
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.29.x-*' kubectl='1.29.x-*' && \
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

Comme pour le master, mettez à jour kubeadm vers la 1.29.1

```shell
{
# replace x in 1.29.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.29.x-*' && \
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
# replace x in 1.29.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.29.x-*' kubectl='1.29.x-*' && \
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
