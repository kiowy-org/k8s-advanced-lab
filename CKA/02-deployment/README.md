# Exercice 2

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Déploiements](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Replicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

</p>
</details>

Dans cet exercice, vous allez créer un Déploiement avec plusieurs replicas. Après avoir inspecté le Déploiement, vous mettrez à jour son modèle de Pod. De plus, vous utiliserez l'historique de déploiement pour revenir à une révision précédente.

1. Créez un Déploiement nommé `nginx` avec 3 replicas. Les Pods doivent utiliser l'image `nginx:1.31.7` dans un conteneur nommé `nginx` (leurs noms sont générés par le contrôleur). Le Déploiement porte le label `tier=backend` dans `metadata.labels`. Utilisez `app=v1` dans `spec.selector.matchLabels` et dans les labels du modèle de Pod.
2. Répertoriez le Déploiement et assurez-vous que le nombre correct de replicas est en cours d'exécution.
3. Mettez à jour l'image en `nginx:1.31.8`.
4. Vérifiez que le changement a été déployé sur toutes les replicas.
5. Mettez à l'échelle le Déploiement à 5 replicas.
6. Consultez l'historique de déploiement du Déploiement.
7. Revenez en arrière sur le Déploiement à la révision 1.
8. Assurez-vous que les Pods utilisent l'image `nginx:1.31.7`.
