# Exercice ETCD

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Machine : `mk8s-master-0-<prenom> (via le bastion)`<br>
* Documentation : [Opérer avec ETCD](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

</p>
</details>

## Sauvegarde

1. Créer dans `default` une ConfigMap témoin `etcd-restore-check` contenant `message=avant-sauvegarde`.
2. Relever l’endpoint client etcd et les chemins de la CA, du certificat client et de sa clé dans les manifests du control plane. Avec les droits nécessaires, utiliser `etcdctl snapshot save` pour écrire `/opt/backup-etcd.db`. Les options TLS sont `--cacert`, `--cert` et `--key` ; ne pas utiliser `--insecure-skip-tls-verify`.
3. Contrôler le snapshot avec `etcdutl snapshot status /opt/backup-etcd.db --write-out=table`.
4. Supprimer uniquement la ConfigMap témoin et vérifier son absence. Cette suppression permet de prouver ensuite la restauration effective.

## Restauration

1. Sauvegarder les manifests du control plane dans un répertoire **hors** de `/etc/kubernetes/manifests`. Conserver le répertoire de données etcd initial pour pouvoir revenir à l’état précédent.
2. Retirer temporairement les manifests de l’API server, du controller-manager et du scheduler du répertoire surveillé par le kubelet. Attendre leur arrêt, observable avec `sudo crictl ps`. Retirer ensuite le manifest etcd et attendre son arrêt. `kubectl` n’est plus utilisable pendant l’arrêt de l’API.
3. Restaurer le snapshot avec `etcdutl snapshot restore` dans le nouveau répertoire `/var/lib/etcd-backup`, qui ne doit pas déjà contenir une restauration. Reprendre les paramètres de membre `--name`, `--initial-cluster` et `--initial-advertise-peer-urls` correspondant au manifest etcd du lab. Consulter la documentation de récupération de la version installée : pour Kubernetes, utiliser le mécanisme de révision majorée et de compaction lorsque pris en charge (`--bump-revision`, `--mark-compacted`) pour invalider les caches des clients.
4. Dans la copie du manifest etcd, modifier le `hostPath.path` du volume de données pour pointer vers `/var/lib/etcd-backup`. Si le chemin monté dans le conteneur reste `/var/lib/etcd`, conserver ce `mountPath` et la valeur correspondante de `--data-dir` : chemin hôte et chemin conteneur sont distincts. Vérifier les permissions du répertoire restauré.
5. Remettre le manifest etcd, attendre la santé de l’endpoint etcd, puis remettre les manifests de l’API server, du controller-manager et du scheduler. Observer les conteneurs et leurs logs si le démarrage échoue.
6. Vérifier le retour de l’API, des nœuds et de la ConfigMap `etcd-restore-check` avec sa valeur `avant-sauvegarde`. Un fichier restauré sur disque sans retour de l’objet témoin ne valide pas le TP.
7. Supprimer la ConfigMap témoin. Conserver les sauvegardes jusqu’à validation par le formateur ; ne pas supprimer le répertoire de données actuellement monté par etcd.

La restauration affecte l’ensemble de ce cluster de laboratoire. Le formateur doit valider la procédure avec les versions effectivement installées avant la session.
