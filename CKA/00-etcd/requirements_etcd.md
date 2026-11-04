# Installer les outils etcd du laboratoire

Vérifier d’abord l’image du membre etcd dans `/etc/kubernetes/manifests/etcd.yaml` et les binaires existants :

```shell
etcdctl version
etcdutl version
uname -m
```

Choisir une release officielle compatible avec la version **majeure et mineure** du serveur etcd du lab, puis vérifier les exigences de restauration de cette release. Ne pas sélectionner automatiquement « latest ». Les téléchargements sont disponibles dans les [releases officielles etcd](https://github.com/etcd-io/etcd/releases).

1. Relever la version exacte approuvée par le formateur (`vX.Y.Z`). Sélectionner l’archive Linux `amd64` pour `x86_64`, ou `arm64` pour `aarch64`. Les VM GCE e2 du lab sont amd64.
2. Télécharger cette archive et son fichier de sommes de contrôle depuis la même release, avec `curl --fail --location --output <fichier> <URL>`. Vérifier la somme SHA-256 de l’archive avant extraction. Arrêter si le téléchargement ou le contrôle échoue.
3. Extraire dans un répertoire temporaire dédié, puis installer les deux binaires avec `sudo install -m 0755 <repertoire-extrait>/etcdctl /usr/local/bin/etcdctl` et la même commande pour `etcdutl`. Ne pas remplacer les fichiers gérés par le système dans `/bin`.
4. Vérifier `/usr/local/bin/etcdctl version` et `/usr/local/bin/etcdutl version`, puis `command -v etcdctl` et `command -v etcdutl`. Employer le chemin absolu si une ancienne version est prioritaire dans le PATH, notamment avec sudo.
5. Vérifier les options disponibles avec `etcdutl snapshot restore --help` et suivre la documentation de récupération de la release choisie.

Les placeholders doivent être remplacés par les valeurs de la release retenue. La présence des outils dans un environnement d’examen doit être vérifiée sur l’hôte indiqué par la tâche ; elle ne remplace pas cette préparation du laboratoire.
