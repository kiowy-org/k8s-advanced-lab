# Exercice 10 — PV/PVC et persistance sur un nœud

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

Documentation : [Volumes persistants](https://kubernetes.io/docs/concepts/storage/persistent-volumes/), [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath).

Ce TP utilise `hostPath` uniquement pour observer le binding et la persistance sur **un worker déterminé** du laboratoire. Le disque n’est pas partagé : `/data/config` sur deux nœuds désigne deux répertoires différents. Déclarer `ReadWriteMany` ne transformerait pas hostPath en stockage partagé.

## Préparation

Utiliser le cluster `k8s` et le namespace dédié `tp10-storage`. Choisir un worker et relever la valeur de son label `kubernetes.io/hostname`. Vérifier que les noms `tp10-pv` et `tp10-pvc` sont libres. Dans l’infrastructure CKA, aucun provisionneur CSI ni StorageClass n’est installé par les scripts fournis ; ce TP utilise un volume statique.

## Exercice

1. Créer le namespace `tp10-storage`. Sur le worker choisi, vérifier que `/data/config/tp10-storage` est un répertoire dédié à cet exercice, sans données à conserver.
2. Créer le PersistentVolume `tp10-pv` avec :
   - `capacity.storage: 512Mi`, `volumeMode: Filesystem`, `accessModes: [ReadWriteOnce]` ;
   - `storageClassName: ""` et `persistentVolumeReclaimPolicy: Retain` ;
   - `hostPath.path: /data/config/tp10-storage` et `hostPath.type: DirectoryOrCreate` ;
   - `nodeAffinity.required` limitant le volume au worker choisi, avec `kubernetes.io/hostname`, l’opérateur `In` et sa valeur réelle.
3. Créer la PVC `tp10-pvc` dans `tp10-storage` avec `requests.storage: 256Mi`, `accessModes: [ReadWriteOnce]`, `storageClassName: ""` et `volumeName: tp10-pv`. Vérifier l’état `Bound` des deux objets et leur liaison.
4. Créer un Pod `app` dans ce namespace, avec l’image nginx retenue pour la session. Monter `tp10-pvc` dans `/var/app/config`. Ajouter un `nodeSelector` sur le même label hostname pour rendre le placement explicite ; ne pas utiliser `nodeName`. Vérifier le nœud réel et les permissions d’écriture.
5. Écrire une donnée témoin avec `kubectl -n tp10-storage exec app -- sh -c 'echo persistant > /var/app/config/temoin.txt'`.
6. Supprimer uniquement le Pod puis le recréer depuis son manifeste, en gardant le même PVC et le même placement. Vérifier que `/var/app/config/temoin.txt` contient toujours `persistant`.
7. Expliquer pourquoi ce test ne démontre ni un stockage partagé entre nœuds ni une haute disponibilité.

`ReadWriteOnce` signifie montage en lecture/écriture depuis un seul nœud, pas nécessairement un seul Pod. Pour hostPath, la capacité déclarée du PV n’impose pas un quota réel de 512Mi sur le répertoire hôte.

## Nettoyage et politique de récupération

Supprimer le Pod puis la PVC. Observer le PV en état `Released` avec la politique `Retain` et vérifier que le fichier reste sur le worker. Supprimer ensuite le PV et le namespace du TP. Supprimer uniquement le fichier témoin et le répertoire dédié, après avoir vérifié qu’ils ne sont plus utilisés. Ne pas effacer `/data/config` dans son ensemble.

Un atelier complémentaire avec un provisionneur CSI pourra comparer provisionnement dynamique, StorageClass et politiques Retain/Delete. Il nécessite une installation supplémentaire ; il n’est pas couvert par ce TP statique.
