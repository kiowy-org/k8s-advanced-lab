# Exercice 9 - Ingress

> [!warning]
> Draft

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Service](https://kubernetes.io/docs/concepts/services-networking/services/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/), [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

</p>
</details>

Dans cet exercice, vous créerez un Ingress pour resoudre l'URL `https://mon-application.worker-0-<prenom>.forma.kiowy.net/`.

1. Créer un déploiement à 1 replica pour l'image `gcr.io/kuar-demo/kuard-amd64:blue` et exposer ses pods via un Service de type `ClusterIP`.
2. Créer un déploiement à 1 replica pour l'image `nginx` et exposer ses pods via un Service de type `ClusterIP`.
3. Créer un Ingress `nginx` nommé `my-app-ing` qui route les chemins `/kuard` et `/nginx` de l'URL `https://mon-application.worker-0-<prenom>.forma.kiowy.net/` respectivement sur les services `kuard` et `nginx`.
4. Utiliser le secret `my-app-tls` pour effectuer la terminaison TLS de cette URL.
