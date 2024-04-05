# Exercice 4

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `default`<br>
* Documentation : [Configurer les probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

</p>
</details>

Dans cet exercice, vous allez créer un Pod exécutant une application NodeJS. Le Pod définira des sondes de disponibilité et de présence avec différents paramètres.


1. Créez un nouveau Pod nommé `hello` avec l'image `bmuschko/nodejs-hello-world:1.0.0` qui expose le port 3000. Fournissez le nom `nodejs-port` pour le port du conteneur.
2. Ajoutez une sonde de disponibilité qui vérifie le chemin d'URL / sur le port référencé avec le nom `nodejs-port` après un délai de 2 secondes. Vous n'avez pas à définir l'intervalle de période.
3. Ajoutez une sonde de présence qui vérifie que l'application est opérationnelle toutes les 8 secondes en vérifiant le chemin d'URL / sur le port référencé avec le nom `nodejs-port`. La sonde devrait commencer après un délai de 5 secondes.
4. Accédez au shell du conteneur et exécutez `curl localhost:3000`. Notez la sortie. Quittez le conteneur.
5. Récupérez les journaux du conteneur. Notez la sortie.