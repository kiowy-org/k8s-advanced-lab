# Exercice 6

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector), [Affinité de Nœud](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity)

</p>
</details>

Dans cet exercice, vous allez planifier un Pod sur un nœud spécifique. Le Pod ne devrait être planifié que sur des nœuds avec l'étiquette ayant la clé `color` et les valeurs attribuées `green` ou `red`.

1. Examinez les nœuds existants et leurs étiquettes attribuées.
2. Choisissez un worker disponible et ordonnançable et étiquetez-le avec la paire clé-valeur `color=green`. Choisissez un deuxième worker disponible et ordonnançable et étiquetez-le avec la paire clé-valeur `color=red`.
3. Définissez un Pod avec l'image `nginx` dans le fichier manifeste YAML `pod.yaml`. Utilisez l'assignation `nodeSelector` pour planifier le Pod sur le nœud avec l'étiquette `color=green`.
4. Créez le Pod et assurez-vous que le nœud correct a été utilisé pour exécuter le Pod.
5. Dans le fichier, retirez le `nodeSelector` initial et ajoutez `nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution` avec une expression sur la clé `color`, l’opérateur `In` et les valeurs `[green, red]`. Un `nodeSelector` conservé se cumulerait avec l’affinité et exclurait toujours red.
6. Supprimez le Pod initial et recréez-le avec le fichier modifié. Changer le fichier ne déplace pas un Pod déjà affecté ; les contraintes de placement de ce Pod ne se modifient pas librement en place.
7. Vérifiez que le Pod est affecté à un nœud portant green ou red. Les deux résultats sont corrects : aucune alternance ni préférence pour red n’est imposée.
