# Exercice 1

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

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

## Vérifier les permissions

Tester en utilisant l’identité `system:serviceaccount:default:johndoe` avec `kubectl auth can-i --as=...`. Le compte qui lance ces vérifications doit être autorisé à usurper cette identité (compte administrateur du lab).

| Action | Portée | Résultat attendu |
| --- | --- | --- |
| `list pods` | `-n default` | yes |
| `list services` | `-n default` | yes |
| `list services` | `--all-namespaces` | yes |
| `list persistentvolumes` | cluster | yes |
| `list pods` | `-n kube-system` | no |
| `create pods` | `-n default` | no |
| `delete services` | `-n default` | no |

Exemple : `kubectl auth can-i list pods -n default --as=system:serviceaccount:default:johndoe`.
Les droits RBAC sont cumulatifs : partir d’un ServiceAccount neuf sans autre binding. `list` et `get` sont deux verbes distincts ; ne pas accorder de permissions supplémentaires pour obtenir les résultats attendus. Il n’est pas nécessaire de créer un Secret de jeton pour ces tests.

## Nettoyage

Supprimer seulement le ServiceAccount, les Roles, ClusterRoles et bindings créés dans cet exercice. Conserver leurs noms dans vos fichiers pour ne pas toucher aux droits d’autres utilisateurs.
