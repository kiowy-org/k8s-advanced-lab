# Exercice 2

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

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

## Validation

Avant l’exercice, vérifier que le Deployment `nginx` n’existe pas dans ce namespace et que les deux images sont téléchargeables sur les nœuds. Ne pas réutiliser un Deployment préexistant : sa révision 1 ne correspondrait pas à ce TP. Le cluster `k8s` du lab convient ; `ek8s` et `wk8s` contiennent déjà un Deployment `nginx`.

Après chaque mise à jour, attendre `kubectl rollout status deployment/nginx --timeout=180s` et vérifier les images des Pods sélectionnés par `app=v1`. Après le rollback, l’image doit être `nginx:1.31.7` et le Deployment doit toujours avoir **5 réplicas** : le rollback restaure le modèle de Pod, pas l’historique du scaling. Le rollback crée une nouvelle révision utilisant l’ancien modèle.

Supprimer le Deployment créé pour ce TP à la fin.
