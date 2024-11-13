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