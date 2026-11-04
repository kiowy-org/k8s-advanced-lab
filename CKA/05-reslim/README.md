# Exercice 5

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

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
3. Pour le conteneur, définissez les `resources.requests` utilisées pour la planification comme suit :

    - CPU : 100m
    - Mémoire : 500Mi
    - Stockage éphémère : 1Gi

4. Pour le conteneur, définissez les `resources.limits` comme suit (sans limite CPU dans cet exercice) :

    - Mémoire : 500Mi
    - Stockage éphémère : 2Gi

5. Créez le Pod à partir du manifeste YAML.
6. Inspectez les détails du Pod. Sur quel nœud le Pod s'exécute-t-il ?

## Interpréter et vérifier

Les requests ne forcent pas une consommation minimale. Vérifier dans le Pod les valeurs CPU `100m`, mémoire `500Mi`, stockage éphémère `1Gi`, puis les limites mémoire `500Mi` et stockage éphémère `2Gi`.

Lire `kubectl get pod hello-world -o jsonpath='{.status.qosClass}'` : la classe attendue est `Burstable`, car le conteneur n’a pas de limite CPU. L’égalité request/limit mémoire seule ne suffit pas pour `Guaranteed`. Vérifier qu’aucun LimitRange du namespace n’ajoute une limite CPU par défaut.

La limite `ephemeral-storage` concerne les logs, la couche inscriptible du conteneur et l’`emptyDir` sur disque ; elle ne constitue pas une réservation de disque ni un `emptyDir.sizeLimit`. Le suivi et l’éviction nécessitent une configuration de stockage local prise en charge par le kubelet. Le montage `/var/log` ne redirige pas automatiquement stdout/stderr vers ce volume.

Pour observer la planification, créer un second manifeste, nommé `hello-world-pending`, avec une request CPU supérieure à l’allocatable de tous les nœuds du lab et sans limite CPU. Le Pod doit rester `Pending` avec un événement `FailedScheduling` indiquant un CPU insuffisant ; le supprimer après observation. Ne pas saturer volontairement les disques ou la mémoire des nœuds.

`kubectl top pod hello-world` est un complément seulement si Metrics Server est installé ; l’infrastructure CKA fournie ne l’installe pas. Nettoyer les deux Pods après l’exercice.
