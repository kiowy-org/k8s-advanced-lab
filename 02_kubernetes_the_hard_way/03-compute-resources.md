# Provisioning Compute Resources

Les ressources nécessaires sont déjà provisionnées pour vos clusters. Voici un résumé des composants à votre disposition : (*remplacez prenom par votre prenom en minuscule*)

| Host                | Private IP  | Public hostname                     |
| ------------------- | ----------- | ----------------------------------- |
| controller-0-prenom | 10.240.0.10 | controller-0-prenom.forma.kiowy.net |
| controller-1-prenom | 10.240.0.11 | controller-1-prenom.forma.kiowy.net |
| controller-2-prenom | 10.240.0.12 | controller-2-prenom.forma.kiowy.net |
| worker-0-prenom     | 10.240.0.20 | worker-0-prenom.forma.kiowy.net     |
| worker-1-prenom     | 10.240.0.21 | worker-1-prenom.forma.kiowy.net     |

Une IP Publique a également été provisionnée pour le load balancer devant l'API Kubernetes. Son adresse est `api.<prenom>.forma.kiowy.net`.

Vos machines peuvent communiquer entre elles dans le réseau privée, le pare feu laisse également la connexion SSH entrante de l'extérieur.

## Vérifier l'accès SSH

Vous aurez besoin d'éxecuter des commandes via ssh sur chaque machines, vérifiez la connexion grâce à la commande :

```
ssh -i id_forma etudiant@controller-0-prenom.forma.kiowy.net
```

Vous devriez arriver sur le prompt ubuntu :
```
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-1042-gcp x86_64)
...
```

Tapez `exit` pour quitter `controller-0` :

```
etudiant@controller-0-prenom:~$ exit
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
