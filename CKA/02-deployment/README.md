# Exercice 2

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Déploiements](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Replicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

Dans cet exercice, vous allez créer un Déploiement avec plusieurs replicas. Après avoir inspecté le Déploiement, vous mettrez à jour son modèle de Pod. De plus, vous utiliserez l'historique de déploiement pour revenir à une révision précédente.


1. Créez un Déploiement nommé `nginx` avec 3 replicas. Les Pods doivent utiliser l'image `nginx:1.23.0` et le nom `nginx`. Le Déploiement utilise l'étiquette `tier=backend`. Le modèle de Pod devrait utiliser l'étiquette `app=v1`.
2. Répertoriez le Déploiement et assurez-vous que le nombre correct de replicas est en cours d'exécution.
3. Mettez à jour l'image en `nginx:1.23.4`.
4. Vérifiez que le changement a été déployé sur toutes les replicas.
5. Attribuez la cause de changement "Récupérer la version de correctif" à la révision.
6. Mettez à l'échelle le Déploiement à 5 replicas.
7. Consultez l'historique de déploiement du Déploiement.
8. Revenez en arrière sur le Déploiement à la révision 1.
9. Assurez-vous que les Pods utilisent l'image `nginx:1.23.0`.