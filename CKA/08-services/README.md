# Exercice 8

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

</p>
</details>

Dans cet exercice, vous allez créer un Déploiement et exposer un port de conteneur pour ses Pods. Vous allez démontrer les différences entre les types de service ClusterIP et NodePort.

1. Créez un Service nommé `myapp` de type `ClusterIP` qui expose le port 80 et est mappé sur le port cible 80, avec le selector `app=myapp`.
2. Créez un Déploiement nommé `myapp` qui crée 1 réplique en exécutant l'image `nginx:1.23.4-alpine`. Exposez le port de conteneur 80 ; le selector du Deployment et les labels de son template sont `app=myapp`.
3. Mettez à l'échelle le Déploiement à 2 répliques.
4. Attendez la disponibilité des deux répliques et inspectez `kubectl get endpointslices -l kubernetes.io/service-name=myapp`. Depuis un Pod temporaire BusyBox, utilisez `wget -qO- -T 10 http://myapp` : BusyBox ne fournit pas curl par défaut. Par exemple : `kubectl run service-client --rm -i --restart=Never --image=busybox:1.36.1 -- wget -qO- -T 10 http://myapp`.
5. Passez le Service en `NodePort` et relevez son `nodePort` ainsi que l’adresse interne d’un worker.
6. Depuis le **bastion** (hors du cluster mais dans son VPC), exécutez `curl --max-time 10 http://10.240.1.20:<nodePort>`. L’infrastructure CKA n’expose pas les workers sur Internet ; leur IP privée n’est pas directement joignable depuis votre ordinateur.
