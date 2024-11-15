# Accès à l'infra

```shell
# Accès au bastion (depuis votre machine)
ssh -i ../id_forma student@cks-<prenom>.forma.kiowy.net

# Accès aux noeuds (depuis le bastion)
ssh training-master-0-<prenom>
ssh training-node-0-<prenom>
ssh training-node-1-<prenom>
```

# Exercice 1

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Best practices](https://kubernetes.io/docs/concepts/configuration/overview/)

</p>
</details>

Dans cet exercice vous devez identifier les mauvaises pratiques de sécurité dans la configuration.


## Vulnérabilités d'un Dockerfile

Dans le Dockerfile suivant, identifiez vous des vulnérabilités ?

```
FROM ubuntu

# Add MySQL configuration
COPY my.cnf /etc/mysql/conf.d/my.cnf
COPY mysqld_charset.cnf /etc/mysql/conf.d/mysqld_charset.cnf

RUN apt-get update && \
    apt-get -yq install mysql-server-5.6 &&

# Add MySQL scripts
COPY import_sql.sh /import_sql.sh
COPY run.sh /run.sh

# Configure credentials
COPY secret-token .                                       
RUN /etc/register.sh ./secret-token                       
RUN rm ./secret-token       

EXPOSE 3306
CMD ["/run.sh"]
```

## Vulnérabilités de la configuration

Dans les manifests suivants, identifiez vous des vulnérabilités ?

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: mycontainer
        image: redis
        command: ["/bin/sh"]
        args:
        - "-c"
        - "echo $SECRET_USERNAME && echo $SECRET_PASSWORD && docker-entrypoint.sh"
        env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        env:
        - name: Username
          value: Administrator
        - name: Password
          value: MyDiReCtP@sSw0rd
        ports:
        - containerPort: 80
          name: web
```

---
# Exercice 2

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Trivy Documentation](https://aquasecurity.github.io/trivy/v0.18.3/)

</p>
</details>

Dans cet exercice, vous allez utiliser Trivy, un scanner de vulnérabilités, pour identifier les CVEs connues dans les images Docker fournies.

## Scanner des images Docker

Sur votre terminal principal, utilisez Trivy pour analyser les images suivantes et identifier les vulnérabilités :

1. `nginx:1.16.1-alpine`
2. `k8s.gcr.io/kube-apiserver:v1.18.0`
3. `k8s.gcr.io/kube-controller-manager:v1.18.0`
4. `docker.io/weaveworks/weave-kube:2.7.0`

Exécutez la commande suivante pour chaque image :

```bash
trivy image <image_name>
```

Remplacez `<image_name>` par le nom de l'image à scanner, puis interprétez les résultats pour identifier les vulnérabilités critiques ou élevées.

---
# Exercice 3

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [SPDX Documentation](https://spdx.dev/)
* Documentation : [CycloneDX Documentation](https://cyclonedx.org/)
* Documentation : [Trivy Documentation](https://aquasecurity.github.io/trivy/v0.18.3/)

</p>
</details>

Dans cet exercice, vous allez manipuler des SBOM (Software Bill of Materials) en utilisant les outils `bom` et `trivy`.

## Génération et Scan de SBOM

1. **Générer un SBOM au format SPDX-Json :**

   Utilisez `bom` pour générer un SBOM au format SPDX-Json pour l'image suivante :
   
   ```bash
   bom generate -o spdx-json --image registry.k8s.io/kube-apiserver:v1.31.0
   ```

2. **Générer un SBOM au format CycloneDX :**

   Utilisez `trivy` pour générer un SBOM au format CycloneDX pour l'image suivante :
   
   ```bash
   trivy image --format cyclonedx --output kube-controller-manager_sbom.cdx.json registry.k8s.io/kube-controller-manager:v1.31.0
   ```

3. **Scanner un SBOM existant pour les vulnérabilités :**

   Utilisez `trivy` pour analyser un SBOM existant afin de détecter les vulnérabilités connues. Par exemple :
   
   ```bash
   trivy sbom kube-controller-manager_sbom.cdx.json
   ```

Interprétez les résultats pour identifier les vulnérabilités critiques ou élevées présentes dans les composants listés.

---
# Exercice 4

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
* Documentation : [ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook)

</p>
</details>

Dans cet exercice, vous allez configurer un `ImagePolicyWebhook` pour interdire l'utilisation d'images contenant un identifiant spécifique (`danger-danger`) dans les conteneurs déployés.

## Configuration d'un AdmissionController pour l'ImagePolicyWebhook

1. **Créer un fichier de configuration AdmissionConfiguration :**

   Créez un fichier `admission-config.yaml` contenant la configuration du `ImagePolicyWebhook` :

   ```yaml
   apiVersion: apiserver.config.k8s.io/v1
   kind: AdmissionConfiguration
   plugins:
   - name: "ImagePolicyWebhook"
     configuration:
       imagePolicy:
         kubeConfigFile: /etc/kubernetes/webhook/webhook.yaml
         allowTTL: 10
         denyTTL: 10
         retryBackoff: 20
         defaultAllow: true
   ```

2. **Configurer l'API Server pour utiliser l'AdmissionConfiguration :**

   Ajoutez les options suivantes au fichier de configuration de l'API Server (souvent `/etc/kubernetes/manifests/kube-apiserver.yaml`) :

   ```yaml
   --admission-control-config-file=/etc/kubernetes/admission-config.yaml
   --enable-admission-plugins=ImagePolicyWebhook
   ```

3. **Vérification du comportement du webhook :**

   Avec cette configuration, le backend `ImagePolicyWebhook` situé dans le service `webhook-backend` du namespace `image-wh` devrait désormais interdire l'utilisation des images contenant `danger-danger` dans leur nom. Toutes les autres images seront autorisées.

Vérifiez le bon fonctionnement en tentant de déployer un conteneur avec une image interdite et en observant le refus d'admission.
---

# Exercice 5

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Falco Rules](https://falco.org/docs/rules/)
* Documentation : [Falco Output Formats](https://falco.org/docs/configuration/#output)

</p>
</details>

Dans cet exercice, vous allez modifier des règles Falco pour détecter des comportements suspects dans les pods en cours d'exécution. Le format de log spécifié s'appliquera uniquement à la sortie de la règle modifiée.

## Modification des règles Falco

1. **Modifier les règles Falco :**

   Ouvrez le fichier de configuration des règles Falco (par défaut `falco_rules.yaml`) et ajoutez ou modifiez une règle pour détecter des comportements suspects dans les conteneurs, avec le format de log spécifique pour cette règle. Par exemple, vous pouvez ajouter une règle pour détecter l'utilisation non autorisée de shells dans des conteneurs :

   ```yaml
   - rule: Unauthorized Shell in Container
     desc: Détecte l'utilisation non autorisée d'un shell dans un conteneur
     condition: evt.type in (execve, execveat) and container and proc.name in (bash, sh)
     output: "Shell non autorisé détecté dans le conteneur (time=%evt.time, container_id=%container.id, container_name=%container.name, user=%user.name)"
     priority: WARNING
   ```

   Dans cet exemple, le format de log pour cette règle inclut l'heure avec les nanosecondes (`%evt.time`), l'ID du conteneur (`%container.id`), le nom du conteneur (`%container.name`), et le nom de l'utilisateur (`%user.name`).

2. **Redémarrer Falco :**

   Après avoir modifié la configuration, redémarrez Falco pour appliquer les nouvelles règles.

Cette configuration permet de détecter les comportements suspects dans les conteneurs et d'appliquer le format de log personnalisé uniquement à la règle concernée. Vérifiez les logs pour observer la détection des événements en temps réel.

---
# Exercice 6

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

</p>
</details>

Dans cet exercice, vous allez configurer un `Deployment` pour rendre le système de fichiers du conteneur immuable, à l'exception du répertoire `/tmp`, qui restera accessible en écriture. Cette configuration vise à empêcher les modifications du système de fichiers même en cas de compromission.

## Modifier le Deployment pour un système de fichiers immuable

1. **Mettre à jour le Deployment pour rendre le système de fichiers en lecture seule :**

   Modifiez la spécification de `immutable-deployment` en ajoutant un `securityContext` pour activer le mode lecture seule du système de fichiers du conteneur, tout en permettant uniquement au répertoire `/tmp` d'être en écriture.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     namespace: team-purple
     name: immutable-deployment
     labels:
       app: immutable-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: immutable-deployment
     template:
       metadata:
         labels:
           app: immutable-deployment
       spec:
         containers:
         - image: busybox:1.32.0
           command: ['sh', '-c', 'tail -f /dev/null']
           imagePullPolicy: IfNotPresent
           name: busybox
   ```

Déployez cette configuration pour garantir que même en cas de compromission, seules les modifications dans le répertoire `/tmp` seront possibles, rendant le reste du système de fichiers immuable.

---
# Exercice 7

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)

</p>
</details>

Dans cet exercice, vous allez configurer l’audit log du cluster pour enregistrer les accès aux ressources sensibles avec différents niveaux de détail en fonction des critères spécifiés.

## Activer l'audit log avec des règles spécifiques

1. **Créer un fichier de configuration d’audit :**

   Créez un fichier `audit-policy.yaml` avec les règles suivantes :

   ```yaml
   apiVersion: audit.k8s.io/v1
   kind: Policy
   rules:
   - level: Metadata
     resources:
     - group: ""
       resources: ["secrets"]
   
   - level: RequestResponse
     userGroups: ["system:nodes"]
   ```

   Dans cette configuration :
   - Les accès aux ressources de type `Secret` seront enregistrés au niveau `Metadata`, ce qui signifie que seules les métadonnées seront incluses dans les logs.
   - Les actions effectuées par les utilisateurs appartenant au groupe `system:nodes` seront enregistrées au niveau `RequestResponse`, incluant ainsi la demande complète et la réponse.

2. **Appliquer la configuration au kube-apiserver :**

   Ajoutez le chemin vers le fichier de configuration d’audit dans les arguments du kube-apiserver (dans le fichier de configuration `/etc/kubernetes/manifests/kube-apiserver.yaml`) :

   ```yaml
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   --audit-log-path=/var/log/kubernetes/audit.log
   ```

3. **Redémarrer le kube-apiserver :**

   Redémarrez le kube-apiserver pour appliquer la configuration d’audit.

Une fois cette configuration en place, le cluster enregistrera les événements d’audit pour les ressources et utilisateurs spécifiés selon le niveau de détail défini, ce qui vous permettra de surveiller les accès aux informations sensibles et les activités des nœuds.

---
# Exercice 8

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

</p>
</details>

Dans cet exercice, vous allez créer des `Secrets` dans le cluster Kubernetes, puis explorer différentes méthodes pour obtenir la valeur en texte clair de ces `Secrets`.

## Création et récupération des Secrets

1. **Créer un Secret :**

   Créez un fichier YAML pour définir un `Secret` avec un mot de passe codé en base64 :

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: my-secret
     namespace: default
   type: Opaque
   data:
     password: bXlwYXNzd29yZA==  # 'mypassword' encodé en base64
   ```

   Appliquez ce fichier pour créer le `Secret` :

   ```bash
   kubectl apply -f my-secret.yaml
   ```

2. **Méthodes pour récupérer la valeur en texte clair :**

   Une fois le `Secret` créé, essayez différentes méthodes pour obtenir la valeur en texte clair :

   - **Afficher le Secret en base64 et le décoder manuellement :**

     ```bash
     kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode
     ```

   - **Utiliser `kubectl describe` (ne montre pas le contenu en clair, mais utile pour voir les détails du Secret) :**

     ```bash
     kubectl describe secret my-secret
     ```

   - **Monter le Secret dans un Pod et lire le contenu :**

     Créez un Pod qui monte le `Secret` comme volume, puis accédez au fichier pour lire la valeur en texte clair.

     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: secret-reader
       namespace: default
     spec:
       containers:
       - name: secret-reader
         image: busybox
         command: ["cat", "/etc/secret-volume/password"]
         volumeMounts:
         - name: secret-volume
           mountPath: "/etc/secret-volume"
           readOnly: true
       volumes:
       - name: secret-volume
         secret:
           secretName: my-secret
     ```

     Appliquez le Pod, puis vérifiez les logs pour obtenir la valeur en clair :

     ```bash
     kubectl apply -f secret-reader-pod.yaml
     kubectl logs secret-reader
     ```

3. **Conclusion :**

   Ces différentes méthodes vous permettent d'explorer comment accéder aux valeurs des `Secrets` en texte clair.
---

# Exercice 9

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
* Documentation : [Namespace Configuration](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

</p>
</details>

Dans cet exercice, vous allez appliquer le standard de sécurité des pods de niveau `baseline` au Namespace `team-red` pour empêcher la création de pods avec des configurations dangereuses. Vous allez ensuite observer les événements pour comprendre pourquoi le pod du déploiement `container-host-hacker` n'est pas recréé.

## Étapes

1. **Configurer le standard de sécurité des pods :**

   Appliquez la politique de sécurité `baseline` au Namespace `team-red` pour restreindre les configurations non sécurisées, comme l'utilisation de volumes `hostPath` sensibles.

   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: team-red
     labels:
       pod-security.kubernetes.io/enforce: baseline
   ```

   Appliquez cette configuration :

   ```bash
   kubectl apply -f namespace-team-red.yaml
   ```

2. **Supprimer le pod du déploiement `container-host-hacker` :**

   Supprimez le pod en cours d'exécution pour que le contrôleur de déploiement essaie de le recréer :

   ```bash
   kubectl delete pod -l app=container-host-hacker -n team-red
   ```

3. **Vérifier les événements du ReplicaSet :**

   Une fois le pod supprimé, le ReplicaSet du déploiement tentera de recréer le pod, mais échouera en raison des nouvelles restrictions de sécurité. Vérifiez les événements du ReplicaSet pour identifier les messages et les raisons du blocage de la création du pod :

   ```bash
   kubectl describe rs -l app=container-host-hacker -n team-red
   ```

4. **Consulter les événements pertinents :**

   Dans la sortie, recherchez les lignes d'événements contenant la raison de l'échec de la recréation du pod, qui devrait être liée à l'application des règles de sécurité `baseline` qui interdisent l'utilisation de volumes `hostPath` non conformes.
---

# Exercice 10

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes RuntimeClass](https://kubernetes.io/docs/concepts/containers/runtime-class/)
* Documentation : [gVisor](https://gvisor.dev/)

</p>
</details>

Dans cet exercice, vous allez configurer une `RuntimeClass` pour utiliser le runtime sécurisé `gVisor` et déployer un pod utilisant cette configuration sur un nœud spécifique.

## Étapes

1. **Créer une RuntimeClass pour gVisor :**

   Créez un fichier `runtimeclass-gvisor.yaml` pour définir la `RuntimeClass` nommée `gvisor` avec le gestionnaire `runsc` :

   ```yaml
   apiVersion: node.k8s.io/v1
   kind: RuntimeClass
   metadata:
     name: gvisor
   handler: runsc
   ```

   Appliquez cette configuration :

   ```bash
   kubectl apply -f runtimeclass-gvisor.yaml
   ```

2. **Créer un Pod utilisant la RuntimeClass :**

   Créez un fichier `gvisor-test-pod.yaml` pour définir un pod dans le Namespace `team-purple`, en spécifiant la `RuntimeClass` `gvisor` et le nœud cible :

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: gvisor-test
     namespace: team-purple
   spec:
     runtimeClassName: gvisor
     nodeSelector:
       kubernetes.io/hostname: cks7262-node2
     containers:
     - name: nginx
       image: nginx:1.27.1
   ```

   Appliquez cette configuration pour déployer le pod :

   ```bash
   kubectl apply -f gvisor-test-pod.yaml
   ```

3. **Vérifier l'exécution du Pod et récupérer la sortie de `dmesg` :**

   Une fois le pod en cours d'exécution, exécutez la commande `dmesg` dans le conteneur pour afficher les messages du système, en particulier ceux liés à `gVisor` :

   ```bash
   kubectl exec -n team-purple gvisor-test -- dmesg
   ```

La sortie de cette commande affichera les messages du noyau et du runtime, vous permettant de vérifier l'initialisation sécurisée du pod avec `gVisor`.
---

# Exercice 11

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Cilium Network Policies](https://docs.cilium.io/en/stable/policy/language/)

</p>
</details>

Dans cet exercice, vous allez utiliser une `CiliumNetworkPolicy` pour restreindre les communications réseau entre les namespaces `team-a` et `team-b` et autoriser uniquement la résolution DNS de `kiowy.io` en sortie.

## Étapes

1. **Créer une CiliumNetworkPolicy pour restreindre les accès réseau :**

   Créez un fichier `cilium-network-policy.yaml` avec les règles de politique réseau suivantes :

   ```yaml
   apiVersion: cilium.io/v2
   kind: CiliumNetworkPolicy
   metadata:
     name: team-b-access-policy
     namespace: team-b
   spec:
     endpointSelector:
       matchLabels: {}
     ingress:
     - fromEndpoints:
       - matchLabels:
           namespace: team-a
     egress:
     - toPorts:
       - ports:
         - port: "53"
           protocol: ANY
         rules:
           dns:
           - matchPattern: "kiowy.io"
   ```

   Dans cette configuration :
   - La section `ingress` permet uniquement aux pods dans le namespace `team-a` de communiquer avec les pods du namespace `team-b`.
   - La section `egress` autorise uniquement le trafic DNS vers `kiowy.io` pour les pods dans `team-b`.

2. **Appliquer la CiliumNetworkPolicy :**

   Appliquez la configuration de la politique réseau :

   ```bash
   kubectl apply -f cilium-network-policy.yaml
   ```

3. **Vérification de la politique :**

   Vérifiez que seuls les pods du namespace `team-a` peuvent communiquer avec ceux de `team-b` et que la résolution DNS de `kiowy.io` fonctionne en sortie pour les pods de `team-b`.

Cette configuration renforce la sécurité en limitant les communications réseau et en appliquant des règles de résolution DNS spécifiques.

---
# Exercice 12

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

</p>
</details>

Dans cet exercice, vous allez confirmer que l'utilisateur `gianna` ne peut pas lire le contenu des `Secrets` dans le cluster et configurer un `ClusterRole` pour lui permettre de créer des `Pods` et `Deployments` dans plusieurs namespaces, facilitant ainsi la réutilisation de cette autorisation dans le futur.

## Étapes

1. **Vérifier les restrictions de lecture de `Secrets` :**

   Utilisez la commande suivante pour examiner les règles RBAC existantes et confirmer que l'utilisateur `gianna` n'a pas la permission de lire les `Secrets` dans le cluster. Si nécessaire, ajustez les règles pour restreindre l'accès :

   ```bash
   kubectl get clusterrolebinding -o yaml | grep -A 10 'subjects:\|- kind: User\n  name: gianna'
   ```

   Assurez-vous qu'aucun `ClusterRole` ou `Role` attaché à `gianna` ne lui accorde la permission `get` sur les ressources de type `Secret`.

2. **Créer un ClusterRole pour autoriser `gianna` à créer des `Pods` et `Deployments` :**

   Créez un `ClusterRole` nommé `pod-deployment-creator` qui permet de créer des `Pods` et `Deployments`. Ensuite, attachez ce `ClusterRole` à `gianna` dans les namespaces concernés à l’aide de `RoleBindings`.

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: pod-deployment-creator
   rules:
   - apiGroups: ["", "apps"]
     resources: ["pods", "deployments"]
     verbs: ["create"]

   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: gianna-create-pods-deployments
     namespace: security
   subjects:
   - kind: User
     name: gianna
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: ClusterRole
     name: pod-deployment-creator
     apiGroup: rbac.authorization.k8s.io

   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: gianna-create-pods-deployments
     namespace: restricted
   subjects:
   - kind: User
     name: gianna
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: ClusterRole
     name: pod-deployment-creator
     apiGroup: rbac.authorization.k8s.io

   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: gianna-create-pods-deployments
     namespace: internal
   subjects:
   - kind: User
     name: gianna
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: ClusterRole
     name: pod-deployment-creator
     apiGroup: rbac.authorization.k8s.io
   ```

3. **Appliquer les configurations :**

   Appliquez le fichier YAML pour créer le `ClusterRole` et les `RoleBindings` dans chaque namespace (`security`, `restricted`, `internal`) :

   ```bash
   kubectl apply -f clusterrolebindings-gianna.yaml
   ```

4. **Vérification des permissions :**

   Testez les permissions en vous connectant avec l'utilisateur `gianna` ou en utilisant `kubectl auth can-i` pour vérifier qu'il peut créer des `Pods` et `Deployments` dans les namespaces spécifiés, mais qu'il ne peut pas accéder aux `Secrets`.
---

# Exercice 13

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

</p>
</details>

Dans cet exercice, vous allez créer un `ServiceAccount` dont le token ne sera pas monté automatiquement dans les pods. Vous allez ensuite créer un pod utilisant ce `ServiceAccount` et monter le token manuellement dans un chemin spécifique.

## Étapes

1. **Créer le ServiceAccount sans montage automatique du token :**

   Créez un fichier `serviceaccount.yaml` avec la spécification du `ServiceAccount` en désactivant le montage automatique du token :

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: custom-sa
   automountServiceAccountToken: false
   ```

   Appliquez cette configuration pour créer le `ServiceAccount` :

   ```bash
   kubectl apply -f serviceaccount.yaml
   ```

2. **Créer un Pod en utilisant le ServiceAccount et en montant manuellement le token :**

   Créez un fichier `pod-with-token.yaml` pour définir un pod qui utilise ce `ServiceAccount` et monte explicitement le token dans le chemin `/tmp/kube/token` :

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: token-mount-pod
   spec:
     serviceAccountName: custom-sa
     containers:
     - name: test-container
       image: busybox
       command: ["sh", "-c", "sleep 3600"]
       volumeMounts:
       - name: token-volume
         mountPath: /tmp/kube/token
         subPath: token
     volumes:
     - name: token-volume
       projected:
         sources:
         - serviceAccountToken:
             path: token
             expirationSeconds: 3600
             audience: api
   ```

   Appliquez la configuration pour créer le pod :

   ```bash
   kubectl apply -f pod-with-token.yaml
   ```

3. **Vérification :**

   Une fois le pod en cours d'exécution, vous pouvez vérifier que le token du `ServiceAccount` est monté dans `/tmp/kube/token` :

   ```bash
   kubectl exec token-mount-pod -- ls /tmp/kube/token
   ```

   ---
   # Exercice 14

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Cluster Upgrade](https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/)

</p>
</details>

Dans cet exercice, vous allez mettre à jour le cluster Kubernetes de la version `1.30.5` à la version `1.31.1`. Assurez-vous d’effectuer chaque étape dans l'ordre pour éviter les interruptions de service.

## Étapes

1. **Vérifier la compatibilité et préparer le cluster pour la mise à jour :**

   - Consultez la documentation de la version `1.31.1` pour vérifier les changements et notes de version.

2. **Mettre à jour le plan de contrôle :**

   Connectez-vous au nœud principal du plan de contrôle et exécutez les commandes de mise à jour kubeadm :

   ```bash
   sudo apt-get update && sudo apt-get install -y kubeadm=1.31.1-00
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.31.1
   ```

   - Cette commande mettra à jour les composants du plan de contrôle (API Server, Controller Manager, Scheduler) vers la version `1.31.1`.
   - Suivez les instructions affichées par kubeadm pour chaque étape.

3. **Mettre à jour `kubelet` et `kubectl` sur chaque nœud du cluster :**

   - Une fois le plan de contrôle mis à jour, mettez à jour `kubelet` et `kubectl` sur chaque nœud.

     ```bash
     sudo apt-get update && sudo apt-get install -y kubelet=1.31.1-00 kubectl=1.31.1-00
     sudo systemctl restart kubelet
     ```

4. **Vérifier la mise à jour :**

   Après la mise à jour de tous les nœuds, vérifiez la version du cluster pour confirmer que la mise à jour a réussi :

   ```bash
   kubectl version --short
   ```
   ---
   # Exercice 15

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

</p>
</details>

Dans cet exercice, vous allez configurer deux `NetworkPolicies` pour contrôler l'accès au service de métadonnées à l'adresse `http://192.168.100.21:32000`. L'objectif est de restreindre l'accès pour tous les pods, sauf pour ceux ayant un rôle spécifique.

## Étapes

1. **Créer une NetworkPolicy pour bloquer l'accès au service de métadonnées :**

   Créez une première `NetworkPolicy` nommée `metadata-deny` pour interdire tout accès sortant vers `192.168.100.21` pour tous les pods du namespace `metadata-access`.

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: metadata-deny
     namespace: metadata-access
   spec:
     podSelector: {}
     policyTypes:
     - Egress
     egress:
     - to:
       - ipBlock:
           cidr: 192.168.100.21/32
         ports:
         - port: 32000
           protocol: TCP
   ```

   Cette configuration bloque toutes les connexions vers `192.168.100.21:32000` pour les pods dans le namespace `metadata-access`, tout en permettant l'accès sortant vers d'autres adresses.

2. **Créer une NetworkPolicy pour autoriser l'accès au service de métadonnées pour certains pods :**

   Créez une seconde `NetworkPolicy` nommée `metadata-allow` pour autoriser uniquement les pods ayant le label `role: metadata-accessor` à accéder à l'endpoint `192.168.100.21`.

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: metadata-allow
     namespace: metadata-access
   spec:
     podSelector:
       matchLabels:
         role: metadata-accessor
     policyTypes:
     - Egress
     egress:
     - to:
       - ipBlock:
           cidr: 192.168.100.21/32
         ports:
         - port: 32000
           protocol: TCP
   ```

   Cette politique permet aux pods portant le label `role: metadata-accessor` dans le namespace `metadata-access` d'accéder à `192.168.100.21:32000`.

3. **Appliquer les NetworkPolicies :**

   Appliquez les fichiers pour créer les politiques réseau :

   ```bash
   kubectl apply -f metadata-deny.yaml
   kubectl apply -f metadata-allow.yaml
   ```

4. **Vérification :**

   Testez les règles en tentant des connexions depuis les pods existants dans le namespace `metadata-access`. Seuls les pods avec le label `role: metadata-accessor` devraient pouvoir accéder à l'endpoint `192.168.100.21:32000`, tandis que les autres pods auront l'accès bloqué.

---
# Exercice 16

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
* Documentation : [Kube-bench](https://github.com/aquasecurity/kube-bench)

</p>
</details>

Dans cet exercice, vous allez utiliser `kube-bench` pour évaluer certaines configurations du cluster Kubernetes par rapport aux recommandations du CIS Benchmark. Vous allez corriger les configurations si nécessaire.

## Étapes

1. **Vérifier et corriger les paramètres du nœud de contrôle :**

   - **Connectez-vous au nœud de contrôle.**

     ```bash
     ssh cks3477@cks3477-controlplane
     ```

   - **Vérifiez l’argument `--profiling` du `kube-controller-manager` :**

     Utilisez `kube-bench` pour vérifier si l’argument `--profiling` est configuré correctement dans le fichier de configuration du `kube-controller-manager`. Cet argument devrait être défini sur `false` pour des raisons de sécurité.

     ```bash
     sudo kube-bench run --check "1.2.10"
     ```

     - Si le test échoue, modifiez le fichier de configuration du `kube-controller-manager` (souvent `/etc/kubernetes/manifests/kube-controller-manager.yaml`) pour définir `--profiling=false`, puis redémarrez le service.

   - **Vérifiez la propriété du répertoire `/var/lib/etcd` :**

     Utilisez `kube-bench` pour vérifier que le répertoire `/var/lib/etcd` appartient bien à `etcd`.

     ```bash
     sudo kube-bench run --check "1.3.2"
     ```

     - Si le test échoue, corrigez les permissions du répertoire :

       ```bash
       sudo chown etcd:etcd /var/lib/etcd
       ```

2. **Vérifier et corriger les paramètres du nœud worker :**

   - **Connectez-vous au nœud worker.**

     ```bash
     ssh cks3477@cks3477-node1
     ```

   - **Vérifiez les permissions du fichier de configuration de kubelet `/var/lib/kubelet/config.yaml` :**

     Utilisez `kube-bench` pour vérifier que les permissions sont définies à `644` pour le fichier de configuration de kubelet.

     ```bash
     sudo kube-bench run --check "4.2.1"
     ```

     - Si le test échoue, modifiez les permissions du fichier pour les aligner sur les recommandations :

       ```bash
       sudo chmod 644 /var/lib/kubelet/config.yaml
       ```

   - **Vérifiez l’argument `--client-ca-file` de kubelet :**

     Utilisez `kube-bench` pour vérifier que l’argument `--client-ca-file` est bien configuré pour sécuriser les connexions du kubelet.

     ```bash
     sudo kube-bench run --check "4.2.2"
     ```

     - Si le test échoue, modifiez la configuration de kubelet pour inclure l'argument `--client-ca-file` pointant vers le fichier CA approprié, puis redémarrez le service kubelet.

3. **Vérification finale :**

   Exécutez de nouveau `kube-bench` sur les nœuds pour confirmer que les corrections sont bien appliquées et que les configurations respectent les recommandations CIS.
---
# Exercice 17

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* Documentation : [Nginx Ingress TLS Configuration](https://kubernetes.github.io/ingress-nginx/user-guide/tls/)

</p>
</details>

Dans cet exercice, vous allez configurer l'Ingress `secure` dans le namespace `team-pink` pour utiliser un certificat TLS spécifique à la place du certificat par défaut généré par le contrôleur Nginx Ingress.

## Étapes

1. **Créer un Secret TLS avec le certificat et la clé fournis :**

   Utilisez les fichiers `/opt/course/15/tls.key` et `/opt/course/15/tls.crt` pour créer un `Secret` TLS dans le namespace `team-pink` :

   ```bash
   kubectl create secret tls secure-tls-cert --key=/opt/course/15/tls.key --cert=/opt/course/15/tls.crt -n team-pink
   ```

   Ce secret nommé `secure-tls-cert` contiendra le certificat et la clé TLS.

2. **Mettre à jour la ressource Ingress pour utiliser le Secret TLS :**

   Modifiez la configuration de l'Ingress `secure` dans le namespace `team-pink` pour référencer le `Secret` créé. Voici un exemple de spécification YAML que vous pouvez appliquer pour la mise à jour :

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: secure
     namespace: team-pink
   spec:
     tls:
     - hosts:
       - secure-ingress.test
       secretName: secure-tls-cert
     rules:
     - host: secure-ingress.test
       http:
         paths:
         - path: /app
           pathType: Prefix
           backend:
             service:
               name: app-service
               port:
                 number: 80
         - path: /api
           pathType: Prefix
           backend:
             service:
               name: api-service
               port:
                 number: 80
   ```

   Appliquez cette configuration pour mettre à jour l'Ingress :

   ```bash
   kubectl apply -f ingress-secure.yaml
   ```

3. **Vérification :**

   Testez l'accès à l'Ingress en utilisant `curl` avec l'option `-k` pour ignorer la vérification du certificat auto-signé :

   - **Pour HTTP :**

     ```bash
     curl -v http://secure-ingress.test:31080/app
     ```

   - **Pour HTTPS :**

     ```bash
     curl -kv https://secure-ingress.test:31443/app
     ```
---
# Exercice 18

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Documentation : [AppArmor in Kubernetes](https://kubernetes.io/docs/tutorials/security/apparmor/)
* Documentation : [Node Labels in Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

</p>
</details>

Dans cet exercice, vous allez installer et activer un profil AppArmor sur un nœud, puis déployer une application avec ce profil, en enregistrant les logs pour analyse.

## Étapes

1. **Connexion au nœud et ajout du label :**

   Connectez-vous au nœud `cks7262-node1` :

   ```bash
   ssh cks7262@cks7262-node1
   ```

   Ajoutez le label `security=apparmor` au nœud :

   ```bash
   kubectl label nodes cks7262-node1 security=apparmor
   ```

2. **Installation du profil AppArmor :**

   Assurez-vous que le profil AppArmor est installé et activé sur le nœud. Vous pouvez utiliser une commande comme celle-ci pour vérifier ou appliquer un profil (le profil doit être spécifié par l’équipe de sécurité) :

   ```bash
   sudo apparmor_parser -r /etc/apparmor.d/path-to-profile
   ```

3. **Créer le Déploiement avec le profil AppArmor :**

   Créez un fichier `apparmor-deployment.yaml` pour configurer le déploiement, en incluant l’annotation AppArmor dans le conteneur nommé `c1` :

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: apparmor
     namespace: default
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: apparmor
     template:
       metadata:
         labels:
           app: apparmor
         annotations:
           container.apparmor.security.beta.kubernetes.io/c1: localhost/path-to-profile
       spec:
         nodeSelector:
           security: apparmor
         containers:
         - name: c1
           image: nginx:1.27.1
   ```

   Appliquez cette configuration :

   ```bash
   kubectl apply -f apparmor-deployment.yaml
   ```

4. **Vérification et enregistrement des logs :**

   Si le pod ne fonctionne pas correctement avec le profil AppArmor activé, accédez aux logs du pod et enregistrez-les pour l’équipe de développement sur le nœud `cks7262` :

   - **Vérifiez l’état du pod :**

     ```bash
     kubectl get pods -l app=apparmor
     ```

   - **Récupérez les logs et enregistrez-les :**

     ```bash
     POD_NAME=$(kubectl get pods -l app=apparmor -o jsonpath="{.items[0].metadata.name}")
     kubectl logs $POD_NAME > /opt/course/9/logs/apparmor-pod.log
     ```
