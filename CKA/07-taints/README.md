# Exercice 7

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Taints et Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

</p>
</details>

Dans cet exercice, vous utiliserez le concept de taints et de tolerations. Tout d'abord, vous allez créer un Pod. Ce Pod sera planifié sur l'un des nœuds. Ensuite, vous ajouterez une taint au nœud sur lequel s'exécute le Pod et définirez un effet de tolérance qui évacue le Pod du nœud.


1. Définissez un Pod avec l'image `nginx` dans le fichier manifeste YAML `pod.yaml`.
2. Créez le Pod et vérifiez sur quel nœud le Pod s'exécute.
3. Ajoutez une taint au nœud. Définissez-la sur `exclusive: yes`.
4. Modifiez l'objet Pod en direct en ajoutant la tolérance suivante : Elle devrait être égale à la paire clé-valeur de la taint et avoir l'effet `NoExecute`.
5. Observez le comportement en cours d'exécution du Pod. Si votre cluster dispose de plus d'un nœud, sur lequel vous attendez-vous à ce que le Pod s'exécute-t-il ?
6. Supprimez la taint du nœud. Vous attendez-vous à ce que le Pod continue à s'exécuter sur le nœud ?