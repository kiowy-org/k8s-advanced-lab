# Exercice ETCD

> [!IMPORTANT]
> Si `etcdctl` & `etcdutl` ne sont pas présents sur le cluster, compléter le prérequis: [Installer les outils ETCD sur le cluster kubeadm](requirements_etcd.md)

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Machine : `controller-0-prenom.forma.kiowy.net`<br>
* Documentation : [Opérer avec ETCD](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

</p>
</details>

Dans cet exercice, vous allez un backup et une restauration de ETCD.

## Backup de ETCD

1. Réaliser un backup de ETCD dans un fichier `/opt/backup-etcd.db`

## Restaurer de ETCD

1. A partir de `/opt/backup-etcd.db`, restaurez ETCD via le **data-dir** `/var/lib/etcd-backup`
