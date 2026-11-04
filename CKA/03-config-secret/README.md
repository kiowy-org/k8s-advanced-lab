# Exercice 3-1

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

Dans cet exercice, vous allez d'abord créer une ConfigMap à partir d'un fichier de configuration YAML en tant que source. Ensuite, vous créerez un Pod, utiliserez la ConfigMap en tant que Volume et inspecterez le fichier projeté dans le conteneur.


1. Inspectez le fichier de configuration YAML nommé [`application.yaml`](./application.yaml).
2. Créez une nouvelle ConfigMap nommée `app-config` avec `--from-file=application.yaml`. La clé créée est `application.yaml` ; les sections YAML internes ne deviennent pas des clés séparées.
3. Créez un Pod nommé `backend-config` qui consomme la ConfigMap en tant que Volume au chemin de montage `/etc/config`. Le conteneur exécute l'image `nginx:1.23.4-alpine`.
4. Accédez au shell du Pod et inspectez le fichier `/etc/config/application.yaml` et comparez son contenu au fichier source.

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
2. Créez un Pod nommé `backend-secret` qui utilise le Secret via `env[].valueFrom.secretKeyRef` pour associer la clé `db-password` à la variable d'environnement `DB_PASSWORD` et exécute le conteneur avec l'image `nginx:1.23.4-alpine`.
3. Accédez au shell du Pod et imprimez les variables d'environnement créées. Vous devriez pouvoir trouver la variable `DB_PASSWORD`.

## Validation et nettoyage

Les deux Pods ont des noms différents et peuvent coexister. Vérifier le contenu avec `kubectl exec backend-config -- cat /etc/config/application.yaml` et la variable fictive avec `kubectl exec backend-secret -- printenv DB_PASSWORD` (résultat : `passwd`).

Un changement de ConfigMap montée peut être projeté avec délai ; l’application doit relire le fichier. Une variable d’environnement issue d’un Secret n’est pas actualisée dans un conteneur déjà lancé : recréer le Pod pour observer une nouvelle valeur. Ne pas tenter de modifier en place ses variables ou ses volumes.

`passwd` est une donnée fictive de laboratoire. Base64 n’est pas du chiffrement. Supprimer les Pods `backend-config` et `backend-secret`, la ConfigMap `app-config` et le Secret `db-credentials` après l’exercice.
