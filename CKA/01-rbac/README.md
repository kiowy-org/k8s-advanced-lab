# Exercice 1

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Utilisation de l'autorisation RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

</p>
</details>

Dans cet exercice vous allez créer des ServiceAccount, Role, ClusterRole et bindings.


## Création du SA

1. Créer un ServiceAccount nommé `johndoe` dans le namespace `default`.

## Droits d'accès

1. Donnez au ServiceAccount `johndoe` le droit de lister les Pods et les Services dans le namespace `default`.
2. Donnez au ServiceAccount `johndoe` le droit de lister les Services dans tout le cluster.
3. Donnez au ServiceAccount `johndoe` le droit de lister les PersistentVolumes dans tout le cluster.