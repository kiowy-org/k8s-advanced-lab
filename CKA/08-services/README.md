# Exercice 8

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

</p>
</details>

Dans cet exercice, vous allez créer un Déploiement et exposer un port de conteneur pour ses Pods. Vous allez démontrer les différences entre les types de service ClusterIP et NodePort.


1. Créez un Service nommé `myapp` de type `ClusterIP` qui expose le port 80 et est mappé sur le port cible 80.
2. Créez un Déploiement nommé `myapp` qui crée 1 réplique en exécutant l'image `nginx:1.23.4-alpine`. Exposez le port de conteneur 80.
3. Mettez à l'échelle le Déploiement à 2 répliques.
4. Créez un Pod temporaire en utilisant l'image `busybox` et exécutez une commande `curl` contre l'IP du service.
5. Modifiez le type de service pour que les Pods puissent être atteints depuis l'extérieur du cluster.
6. Exécutez une commande `curl` contre le service depuis l'extérieur du cluster.