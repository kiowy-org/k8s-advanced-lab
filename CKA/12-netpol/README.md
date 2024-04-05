# Exercice 12

<details>
<summary><b>Référence Rapide</b></summary>
<p>

* Espace de noms : `network`<br>
* Documentation : [Dépannage des Applications](https://kubernetes.io/docs/tasks/debug/debug-application/)

</p>
</details>

Dans cet exercice, vous allez créer une politique réseau permettant à l'application web de interroger sa base de données. La pile d'application se compose d'une application web implémentée en utilisant node.js et d'une base de données MySQL. L'application web se connecte à la base de données lorsqu'elle demande son point de terminaison. L'application web et la base de données MySQL s'exécutent dans un Pod. Les deux Pods ont été exposés par un Service. Le Service pour le Pod de l'application web est de type `NodePort`. Le Service pour la base de données MySQL est de type `ClusterIP`. Par défaut, tout le trafic d'entrée et de sortie n'est pas autorisé dans cet espace de noms.

Le diagramme de flux suivant montre l'architecture générale.

```mermaid
flowchart LR;
    curl-- 192.168.56.10:NodePort -->was{{web-app-service}};
    was-.->wa[web-app];
    wa-- :3306 -->ms{{mysql-service}};
    ms-.->md[(mysql-db)];
```

## Correction du problème dans l'espace de noms "network"

1. Créez un nouvel espace de noms nommé `network`.
2. Dans l'espace de noms, configurez la configuration `kubectl -n network apply -f setup.yaml`.
3. Liste tous les objets et assurez-vous que leur état affiche `Ready`.
4. Le Pod exécutant l'application web expose le port de conteneur 3000. Depuis votre machine, exécutez `curl` ou `wget` pour accéder à l'application via le point de terminaison du Service depuis l'extérieur du cluster. Une réponse réussie devrait afficher `Successfully connected to database!`, une réponse d'échec devrait afficher `Failed to connect to database: <message d'erreur>`.
5. Ajoutez une nouvelle NetworkPolicy permettant à l'application web de se connecter à mysql-db uniquement sur le port 3306.
6. La commande `curl` ou `wget` devrait maintenant afficher le message `Successfully connected to database!`.
7. Supprimez l'espace de noms `network`.