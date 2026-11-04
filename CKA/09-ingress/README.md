# Exercice 9 — Ingress HTTPS avec Traefik

Documentation : [installation Traefik](https://doc.traefik.io/traefik/setup/kubernetes/), [Ingress Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/).

## Préparation par le formateur

L’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka) fournit des workers privés et un bastion SSH, sans contrôleur Ingress ni DNS applicatif. Utiliser le cluster `k8s`, depuis son control plane `k8s-master-0-<prenom>` via le bastion `cka-<prenom>.forma.kiowy.net` et son kubeconfig local. Ne pas employer un kubeconfig fusionné dont seules les entrées de contexte ont été renommées.

1. Installer Helm sur le control plane si nécessaire. Relever `kubectl version` et choisir une version du chart officiel Traefik compatible avec ce serveur. Le code d’infrastructure installe Kubernetes 1.29 : une mise à niveau du lab est à prévoir ; ne pas supposer qu’un chart récent est compatible. Consigner la version exacte approuvée pour la session dans `TRAEFIK_CHART_VERSION`.
2. Ajouter le dépôt puis installer une release dédiée au TP :

```shell
helm repo add traefik https://traefik.github.io/charts
helm repo update
# Définir TRAEFIK_CHART_VERSION avec la version exacte validée par le formateur.
: "${TRAEFIK_CHART_VERSION:?Version du chart validée requise}"
helm upgrade --install tp09-traefik traefik/traefik   --version "$TRAEFIK_CHART_VERSION"   --namespace tp09-traefik --create-namespace   --set service.type=NodePort   --set ingressClass.name=tp09-traefik   --set ingressClass.isDefaultClass=false   --set providers.kubernetesIngress.enabled=true   --set providers.kubernetesIngress.ingressClass=tp09-traefik   --wait --timeout 180s
```

3. Vérifier le Deployment, le Service et `kubectl get ingressclass tp09-traefik -o yaml`. Relever le NodePort HTTPS (`websecure`) du Service de cette release. Depuis le bastion, vérifier que ce port est joignable sur `10.240.1.20`. Aucun LoadBalancer public ni ouverture de pare-feu Internet n’est requis.
4. Créer le namespace `tp09-ingress`. Choisir les noms locaux `kuard.tp09.test` et `nginx.tp09.test`. Générer un certificat autosigné limité au laboratoire, avec les deux noms en SAN :

```shell
mkdir -p tp09-tls
chmod 700 tp09-tls
openssl req -x509 -nodes -newkey rsa:2048 -days 7   -keyout tp09-tls/tls.key -out tp09-tls/tls.crt   -subj '/CN=nginx.tp09.test'   -addext 'subjectAltName=DNS:nginx.tp09.test,DNS:kuard.tp09.test'
kubectl -n tp09-ingress create secret tls my-app-tls   --key=tp09-tls/tls.key --cert=tp09-tls/tls.crt
```

Fournir aux participants le certificat public `tls.crt`, le port HTTPS et les images applicatives épinglées validées pour la session. Garder la clé privée sur la machine de préparation.

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
