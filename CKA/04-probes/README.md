# Exercice 4

## Environnement du TP

Sur l’infrastructure [infra-formations/cka](https://github.com/kiowy-org/infra-formations/tree/main/cka), rejoindre le bastion `student@cka-<prenom>.forma.kiowy.net`, puis le control plane `student@k8s-master-0-<prenom>` et utiliser son kubeconfig local (`~/.kube/config`). Vérifier `kubectl get nodes -o wide` avant de commencer. Les deux workers de ce cluster sont `k8s-node-0-<prenom>` et `k8s-node-1-<prenom>`.

Le kubeconfig fusionné du bastion doit avoir des noms de clusters **et** d’utilisateurs distincts pour chaque contexte ; renommer seulement les contextes ne suffit pas. Tant que cette vérification n’est pas faite, utiliser l’accès direct au control plane décrit ci-dessus.

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Configurer les probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

</p>
</details>

Dans cet exercice, vous allez créer un Pod exécutant une application NodeJS. Le Pod définira des sondes de disponibilité (readiness) et de vivacité (liveness) avec différents paramètres.


1. Préparez le manifeste d’un Pod nommé `hello` avec l'image `bmuschko/nodejs-hello-world:1.0.0` qui expose le port 3000. Fournissez le nom `nodejs-port` pour le port du conteneur.
2. Ajoutez une sonde de disponibilité qui vérifie le chemin d'URL / sur le port référencé avec le nom `nodejs-port` après un délai de 2 secondes. Vous n'avez pas à définir l'intervalle de période.
3. Ajoutez une sonde de vivacité qui vérifie que l'application est opérationnelle toutes les 8 secondes en vérifiant le chemin d'URL / sur le port référencé avec le nom `nodejs-port`. La sonde devrait commencer après un délai de 5 secondes.
4. Créez le Pod une fois les deux probes définies. Attendez `kubectl wait --for=condition=Ready pod/hello --timeout=120s`. Dans un terminal, lancez `kubectl port-forward pod/hello 3000:3000`. Depuis un second terminal sur la même machine, lancez `curl --max-time 10 http://127.0.0.1:3000/`, puis arrêtez le port-forward. Cela ne nécessite pas curl dans l’image applicative.
5. Récupérez les journaux du conteneur. Notez la sortie.

## Observer les différences

Préparer des copies du manifeste avec des noms différents, en ne changeant qu’une probe à la fois. Choisir un port non écouté, par exemple 3001, pour provoquer un échec HTTP reproductible.

- Readiness en échec, liveness correcte : le conteneur reste lancé mais le Pod n’est pas Ready. Une readinessProbe n’est pas une cause de redémarrage. Un Service standard n’envoie pas de trafic à ce backend non prêt.
- Liveness en échec, readiness correcte : observer les événements et l’augmentation du compteur `RESTARTS` après les échecs successifs.
- Variante startup : ajouter une `startupProbe` HTTP sur `/` et `nodejs-port`, avec `periodSeconds: 2` et `failureThreshold: 30`. Observer que readiness et liveness attendent son succès. Refaire un essai sur le port non écouté pour observer le redémarrage après échec de startup.

Lire `kubectl describe pod <nom>` et `kubectl logs <nom> --previous` après un redémarrage. Ne pas modifier les probes d’un Pod existant en place : supprimer/recréer chaque Pod de test. Nettoyer tous les Pods créés.

Avant la session, vérifier le téléchargement et l’exécution de l’image applicative sur l’architecture du lab.
