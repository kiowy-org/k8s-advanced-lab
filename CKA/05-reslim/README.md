# Exercice 5

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Gestion des Ressources pour les Pods et les Conteneurs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

</p>
</details>

Vous avez pour tâche de créer un Pod pour exécuter une application dans un conteneur. Pendant le développement de l'application, vous avez effectué un test de charge pour déterminer la quantité minimale de ressources nécessaire et la quantité maximale de ressources que l'application est autorisée à consommer. Définissez ces demandes et limites de ressources pour le Pod.


1. Définissez un Pod nommé `hello-world` exécutant l'image de conteneur `bmuschko/nodejs-hello-world:1.0.0`. Le conteneur expose le port 3000.
2. Ajoutez un Volume de type `emptyDir` et montez-le sur le chemin du conteneur `/var/log`.
3. Pour le conteneur, spécifiez le nombre minimum de ressources comme suit :

    - CPU : 100m
    - Mémoire : 500Mi
    - Stockage éphémère : 1Gi

4. Pour le conteneur, spécifiez le nombre maximum de ressources comme suit :

    - Mémoire : 500Mi
    - Stockage éphémère : 2Gi

5. Créez le Pod à partir du manifeste YAML.
6. Inspectez les détails du Pod. Sur quel nœud le Pod s'exécute-t-il ?