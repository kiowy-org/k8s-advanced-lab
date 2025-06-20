# Exercice 10

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Volumes Persistants](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

</p>
</details>

Dans cet exercice, vous allez créer un PersistentVolume, le connecter à une PersistentVolumeClaim et monter le claim sur un chemin spécifique d'un Pod.


1. Créez un PersistentVolume nommé `pv`, en mode d'accès `ReadWriteMany`, avec une capacité de stockage de 512Mi et le chemin d'accès hôte `/data/config`.
2. Créez une PersistentVolumeClaim nommée `pvc`. La request devrait demander 256Mi et utiliser une valeur de chaîne vide pour la classe de stockage. Assurez-vous que la PersistentVolumeClaim est correctement liée après sa création.
3. Montez la PersistentVolumeClaim à partir d'un nouveau Pod nommé `app` avec le chemin `/var/app/config`. Le Pod utilise l'image `nginx:1.21.6`.
4. Ouvrez un shell interactif dans le Pod et créez un fichier dans le répertoire `/var/app/config`.
