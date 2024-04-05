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
2. Choisissez un nœud disponible et étiquetez-le avec la paire clé-valeur `color=green`. Choisissez un deuxième nœud et étiquetez-le avec la paire clé-valeur `color=red`.
3. Définissez un Pod avec l'image `nginx` dans le fichier manifeste YAML `pod.yaml`. Utilisez l'assignation `nodeSelector` pour planifier le Pod sur le nœud avec l'étiquette `color=green`.
4. Créez le Pod et assurez-vous que le nœud correct a été utilisé pour exécuter le Pod.
5. Modifiez la définition du Pod pour le planifier sur des nœuds avec l'étiquette `color=green` ou `color=red`.
6. Vérifiez que le Pod s'exécute sur le nœud correct.