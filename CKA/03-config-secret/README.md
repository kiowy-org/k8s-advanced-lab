# Exercice 3-1

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

Dans cet exercice, vous allez d'abord créer une ConfigMap à partir d'un fichier de configuration YAML en tant que source. Ensuite, vous créerez un Pod, utiliserez la ConfigMap en tant que Volume et inspecterez les paires clé-valeur sous forme de fichiers.


1. Inspectez le fichier de configuration YAML nommé [`application.yaml`](./application.yaml).
2. Créez une nouvelle ConfigMap nommée `app-config` à partir de ce fichier.
3. Créez un Pod nommé `backend` qui consomme la ConfigMap en tant que Volume au chemin de montage `/etc/config`. Le conteneur exécute l'image `nginx:1.23.4-alpine`.
4. Accédez au shell du Pod et inspectez le fichier au chemin du Volume monté.

---

# Exercice 3-2

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

</p>
</details>

Dans cet exercice, vous allez d'abord créer un Secret à partir de valeurs littérales. Ensuite, vous créerez un Pod et consommerez le Secret sous forme de variables d'environnement. Enfin, vous imprimerez ses valeurs depuis le conteneur.


1. Créez un nouveau Secret nommé `db-credentials` avec la paire clé/valeur `db-password=passwd`.
2. Créez un Pod nommé `backend` qui utilise le Secret en tant que variable d'environnement nommée `DB_PASSWORD` et exécute le conteneur avec l'image `nginx:1.23.4-alpine`.
3. Accédez au shell du Pod et imprimez les variables d'environnement créées. Vous devriez pouvoir trouver la variable `DB_PASSWORD`.