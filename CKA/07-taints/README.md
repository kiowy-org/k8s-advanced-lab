# Exercice 7

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `tp07-taints`<br>
* Documentation : [Taints et Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

</p>
</details>

## 1. NoSchedule : placement des nouveaux Pods

1. Créer un Pod `sans-tolerance` avec l’image nginx retenue pour la session et un `nodeSelector` sur le label `kubernetes.io/hostname` d’un des workers. Ne pas utiliser `nodeName`, qui contournerait le scheduler. Attendre qu’il soit Ready.
2. Ajouter sur ce worker la taint `exclusive=yes:NoSchedule` avec `kubectl taint node <worker> exclusive=yes:NoSchedule`.
3. Vérifier que le Pod existant continue de fonctionner. Créer un deuxième Pod `nouveau` avec le même placement et sans tolérance : il doit rester Pending. Lire les événements de scheduling.
4. Supprimer le Pod Pending puis le recréer avec cette tolérance dans `spec.tolerations` :

```yaml
- key: exclusive
  operator: Equal
  value: "yes"
  effect: NoSchedule
```

5. Vérifier son démarrage. Supprimer les deux Pods et retirer uniquement la taint créée avec `kubectl taint node <worker> exclusive=yes:NoSchedule-`.

## 2. NoExecute : effet sur les Pods existants

1. Créer deux nouveaux Pods nus sur ce même worker, `non-tolerant` et `tolerant`, avec le même `nodeSelector`. Ajouter **avant leur création** à `tolerant` une tolérance `Equal`, clé `exclusive`, valeur `"yes"`, effet `NoExecute`, sans `tolerationSeconds`. Ne pas ajouter cette tolérance à `non-tolerant`. Attendre qu’ils soient Ready.
2. Ajouter `exclusive=yes:NoExecute` au worker. Observer les événements : le Pod non tolérant est évincé ; le Pod tolérant reste. C’est la taint non tolérée qui provoque l’éviction.
3. Constater qu’un Pod nu évincé n’est pas automatiquement recréé ailleurs. Retirer uniquement `exclusive=yes:NoExecute` et supprimer les Pods de cet essai.

## 3. Remplacement par un contrôleur

1. Créer un Deployment `taint-demo` à une réplique, sans tolérance à `exclusive`, avec une affinité requise limitant le placement aux deux workers du lab (`kubernetes.io/hostname` avec `In`). Ne pas le fixer à un seul worker. Attendre son démarrage et relever le nom et l’UID du Pod.
2. Ajouter `exclusive=yes:NoExecute` au worker qui héberge ce Pod. Vérifier que le Deployment crée un **nouveau** Pod, avec un nouvel UID, sur l’autre worker éligible. Le Pod original ne migre pas.
3. Retirer la taint. Vérifier qu’elle ne provoque pas un retour automatique du nouveau Pod sur le premier worker.
4. Variante : recommencer avec une tolérance `NoExecute` et `tolerationSeconds: 30` dans le template du Deployment. Observer le délai d’éviction et expliquer pourquoi une tolérance n’impose aucun nœud de destination.

## Nettoyage

Retirer les seules taints `exclusive` ajoutées pendant ce TP sur les deux workers. Supprimer le namespace `tp07-taints`. Vérifier les taints finales et la santé des autres workloads du lab.
