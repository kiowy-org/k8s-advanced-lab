# Exercice 10

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `tp10-storage`<br>
* Documentation : [Volumes Persistants](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

</p>
</details>

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
