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

Commencons par mettre en place le TP, nous allons créer un compte de service, afin de simuler un utilisateur non administrateur du cluster.

```shell
kubectl create namespace psp-example
kubectl create serviceaccount -n psp-example fake-user
kubectl create rolebinding -n psp-example fake-editor --clusterrole=edit --serviceaccount=psp-example:fake-user
```

Enregistre des alias pour simplifier les commandes suivantes :
```shell
alias kubectl-admin='kubectl -n psp-example'
alias kubectl-user='kubectl --as=system:serviceaccount:psp-example:fake-user -n psp-example'
```

Créez la PodSecurityPolicy **avec kubectl-admin !** (`kubectl-admin create -f <psp.yaml>`).
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: example
spec:
  privileged: false  # Don't allow privileged pods!
  # The rest fills in some required fields.
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

Avec l'utilisateur non admin, créez un Pod simple :
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

Quel retour obtenez vous de l'API ? Pourquoi ?

Vérifiez via la commandes `kubectl-user auth can-i use podsecuritypolicy/example`.

Bindez un role permettant l'utilisation de la PSP à notre utilisateur non admin :
```shell
kubectl-admin create role psp:unprivileged \
    --verb=use \
    --resource=podsecuritypolicy \
    --resource-name=example

kubectl-admin create rolebinding fake-user:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:fake-user

kubectl-user auth can-i use podsecuritypolicy/example
```

Puis réessayez de créer un Pod :
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

Enfin, tentez de créer un Pod privilégié :
```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: privileged
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
      securityContext:
        privileged: true
EOF
```

Supprimez le pod pause : `kubectl-user delete pod pause`

## 4. Délégation de droits des PSP

Tentez de créer un Deployment pour exécuter votre conteneur :

`kubectl-user create deployment pause --image=k8s.gcr.io/pause`

Attendez quelques secondes, puis vérifiez les pods présents `kubectl-user get pods`.

Que constatez vous ? Utilisez `kubectl-user get events | head -n 2` pour comprendre ce qui se passe.

Pour résoudre ce problème, nous devons autoriser le compte de service du Pod à utiliser la PSP.

Vous pouvez créer un RoleBinding avec le compte de service par défaut :
```shell
kubectl-admin create rolebinding default:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:default
```

Vérifiez que vos pods sont bien créés (patientez quelques secondes que le manager puisse réessayer) :
```shell
kubectl-user get pods --watch
```
