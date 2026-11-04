# Exercice 9 - Ingress

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `tp09-ingress`<br>
* Documentation : [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Service](https://kubernetes.io/docs/concepts/services-networking/services/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/), [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

</p>
</details>

Dans cet exercice, vous créerez un Ingress HTTPS. Le formateur fournit Traefik, la classe `tp09-traefik`, le Secret `my-app-tls`, le certificat public et le NodePort HTTPS.

## Exercice

1. Dans `tp09-ingress`, créer un Deployment `kuard` à une réplique avec `gcr.io/kuar-demo/kuard-amd64:blue`, puis un Service ClusterIP `kuard` vers le port 8080. Vérifier au préalable la disponibilité de cette image amd64 sur les VM e2 du lab.
2. Créer un Deployment `nginx` à une réplique avec l’image nginx retenue pour la session et un Service ClusterIP `nginx` vers le port 80. Vérifier les selectors et les EndpointSlices des deux Services.
3. Créer un Ingress `my-app-ing` (`networking.k8s.io/v1`) avec `ingressClassName: tp09-traefik`. Router le chemin `/`, `pathType: Prefix`, du host `kuard.tp09.test` vers `kuard:8080` et celui de `nginx.tp09.test` vers `nginx:80`.
4. Dans `spec.tls`, référencer `my-app-tls` et les deux noms. Ajouter les annotations Traefik `traefik.ingress.kubernetes.io/router.entrypoints: websecure` et `traefik.ingress.kubernetes.io/router.tls: "true"`.
5. Tester depuis le bastion, avec le certificat public disponible localement et le NodePort HTTPS relevé :

```shell
curl --max-time 10 --cacert tls.crt   --resolve nginx.tp09.test:<port-https>:10.240.1.20   https://nginx.tp09.test:<port-https>/
curl --max-time 10 --cacert tls.crt   --resolve kuard.tp09.test:<port-https>:10.240.1.20   https://kuard.tp09.test:<port-https>/
```

Les placeholders `<port-https>` doivent être remplacés par le même port réel partout. `--resolve` fournit la résolution locale tout en conservant le hostname HTTP et le SNI TLS. Les deux pages doivent répondre et la vérification TLS doit réussir sans `-k`.

Les routes par hostname évitent une réécriture de préfixe spécifique au contrôleur : `Prefix /nginx` ne supprimerait pas ce préfixe avant de transmettre la requête à nginx.

## Nettoyage

Supprimer le namespace `tp09-ingress`, désinstaller uniquement la release `tp09-traefik` de ce TP puis supprimer son namespace. Supprimer les fichiers de certificat et clé dédiés après l’exercice. Si la release est partagée entre participants, son nettoyage appartient au formateur en fin de session.
