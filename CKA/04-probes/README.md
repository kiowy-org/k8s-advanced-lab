# Exercice 4

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
