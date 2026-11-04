# Exercice 6

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

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

## Validation et nettoyage

Comparer `kubectl get pods -o wide` et `kubectl get nodes -L color`. Dans le second cas, l’éligibilité des deux workers ne garantit pas que le Pod change de nœud.

Avant de poser les labels, noter leurs valeurs existantes. Après le TP, supprimer le Pod et restaurer ces valeurs ; retirer `color` uniquement s’il était absent initialement. Ne pas retirer les taints du control plane pour obtenir un deuxième worker : utiliser le cluster `k8s` à deux workers.
