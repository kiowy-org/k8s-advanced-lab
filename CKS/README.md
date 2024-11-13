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
   trivy sbom --input kube-controller-manager_sbom.cdx.json
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

