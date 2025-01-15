# Pod SecurityContext et PodSecurityPolicy

Dans ce TP, nous allons manipuler le securityContext des pods, et créer des PodSecurityPolicy.

## 1. Security Context

Commencons par créer un Pod avec un securityContext spécifiant l'utilisateur d'exécution du container :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

Créez ce Pod avec `kubectl apply`. Démarrez un shell dans le conteneur avec `kubectl exec -it security-context-demo -- sh`.

Dans le Pod, utilisez `ps` pour voir les processus, vérifiez l'utilisateur qui exécute les processus.

Toujours dans le Pod, allez dans le dossier `/data` et exécutez la commande `ls -l`, vérifiez l'ID du groupe du dossier `demo/`.

Exécutez les commandes suivantes :
```shell
cd demo
echo hello > testfile
```

Exécutez de nouveau `ls -l` (dans le dossier `demo/`). Vérifiez le groupe et le propriétaire de `testfile`.

Enfin, exécutez la commande `id` pour vérifier vos IDs d'utilisateur.

Vous pouvez également créer un Pod, avec un securityContext pour le Pod et son Container :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-demo-2
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```

Créez ce Pod avec `kubectl apply`, vérifier qui est bien dans l'état Running.

Ouvrez un shell dans le Container via `kubectl exec -it security-context-demo-2 -- sh`.

Dans le Container, listez les processus avec `ps aux`, vérifier l'id d'utilisateur qui exécute les processus, puis quitter le shell.

## 2. capabilities

Créez un second Pod avec le yaml suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-3
spec:
  containers:
  - name: sec-ctx-3
    image: gcr.io/google-samples/node-hello:1.0
```

Ouvrez un shell (`kubectl exec -it security-context-demo-3 -- sh`) et listez les processus (`ps aux`). Vérifiez l'ID de l'user en train d'exécuter les processus.

Exécutez la commande suivante pour voir les capabilities du container :
```shell
cd /proc/1
cat status
```
Notez la valeur retournée et quittez le conteneur.

Créez le Pod suivant, et effectuez la même opération que ci-dessus pour voir les capabilities du container. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

Que constatez vous ?

La liste des constantes des capabilities du noyau linux sont disponibles dans [capability.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/capability.h).


## 3. PodSecurityPolicy, mise en place

**Attention** : seules les sections **3** et **4** ont été modifiées pour parler du Pod Security Standard à la place de PodSecurityPolicy. Les sections **1** et **2** demeurent inchangées.

---

## 3. Pod Security Standard, mise en place

### Configuration du niveau de sécurité sur le namespace

Dans les dernières versions de Kubernetes, la fonctionnalité PodSecurityPolicy est dépréciée et remplacée par le **Pod Security Admission** qui se base sur le **Pod Security Standard** (PSS). Celui-ci repose sur un système de labels appliqués aux namespaces.  

Il existe trois niveaux principaux définis par le standard : `privileged`, `baseline` et `restricted`.  
Pour cet exercice, nous allons commencer par imposer le niveau `restricted` sur le namespace `default` :

```shell
kubectl label namespace default \
  pod-security.kubernetes.io/enforce=restricted \
  --overwrite
```

Ce label impose des contraintes de sécurité plus strictes pour tous les Pods créés dans ce namespace.

### Création d’un Pod simple

Tentons de créer un Pod tout simple :

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
EOF
```

- **Quel retour obtenez-vous de l’API ?**  
- **Pourquoi ?**

Selon la configuration `restricted`, si votre Pod ne demande pas de privilèges particuliers, il devrait pouvoir être créé. Toutefois, si vous aviez tenté d’y ajouter des options non conformes (par exemple un conteneur privilégié), vous auriez vu un message d’erreur indiquant que la demande est bloquée par la politique de sécurité.

### Création d’un Pod privilégié

Pour illustrer le blocage des Pods privilégiés dans le niveau `restricted`, essayons de re-créer les Pods des questions 1 et 2.

Vérifiez la réponse de l’API.  
- Pourquoi ce Pod est-il refusé ?  
- Qu’est-ce qui, dans la politique de `restricted`, empêche ce Pod d’être créé ?

---

## 4. Délégation et effets sur les contrôleurs (Deployment, etc.)

Créons un déploiement qui exécute le même conteneur :

```shell
kubectl-user create deployment pause --image=k8s.gcr.io/pause
```

Patientez quelques secondes, puis vérifiez les Pods dans le namespace :

```shell
kubectl-user get pods
```

- **Que constatez-vous ?**  
- Utilisez `kubectl-user get events | head -n 2` pour comprendre ce qui se passe.

Si le Pod créé par le contrôleur de déploiement ne respecte pas le niveau `restricted`, il sera bloqué.

Selon le niveau de sécurité appliqué, ou s’il y a d’autres contraintes (policy de réseau, etc.), le déploiement peut voir ses Pods refusés.

---
> Pour plus de détails, reportez-vous à la documentation officielle :  
> [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
