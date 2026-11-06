# Exercice 2

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Déploiements](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Replicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

Dans cet exercice, vous allez créer un Déploiement avec plusieurs replicas. Après avoir inspecté le Déploiement, vous mettrez à jour son modèle de Pod. De plus, vous utiliserez l'historique de déploiement pour revenir à une révision précédente.


1. Créez un Déploiement nommé `nginx` avec 3 replicas. Les Pods doivent utiliser l'image `nginx:1.31.7` et le nom `nginx`. Le Déploiement utilise le label `tier=backend`. Le modèle de Pod devrait utiliser le label `app=v1`.
2. Listez le Déploiement et assurez-vous que le nombre correct de replicas est en cours d'exécution.
3. Mettez à jour l'image en `nginx:1.31.8`.
4. Vérifiez que le changement a été déployé sur toutes les replicas.
6. Mettez à l'échelle le Déploiement à 5 replicas.
7. Consultez l'historique de déploiement du Déploiement.
8. Revenez en arrière sur le Déploiement à la révision 1.
9. Assurez-vous que les Pods utilisent l'image `nginx:1.31.7`.
