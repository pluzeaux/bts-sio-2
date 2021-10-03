# SQL Injection

## Qu'est-ce que l'injection SQL (SQLi) ?

![Injection SQL illustration 1]("https://github.com/pluzeaux/bts-sio/blob/main/images/figure_1.png")

L'injection SQL est une vulnérabilité de sécurité Web qui permet à un attaquant d'interférer avec les requêtes qu'une application effectue sur sa base de données. Il permet généralement à un attaquant de visualiser des données qu'il n'est normalement pas en mesure de récupérer. Cela peut inclure des données appartenant à d'autres utilisateurs ou toute autre donnée à laquelle l'application elle-même est en mesure d'accéder. Dans de nombreux cas, un attaquant peut modifier ou supprimer ces données, provoquant des modifications persistantes du contenu ou du comportement de l'application.
Dans certaines situations, un attaquant peut intensifier une attaque par injection SQL pour compromettre le serveur sous-jacent ou une autre infrastructure principale, ou effectuer une attaque par déni de service.

## Quel est l'impact d'une attaque par injection SQL réussie ?

Une attaque par injection SQL réussie peut entraîner un accès non autorisé à des données sensibles, telles que des mots de passe, des détails de carte de crédit ou des informations personnelles sur l'utilisateur. De nombreuses violations de données très médiatisées ces dernières années ont été le résultat d'attaques par injection SQL, entraînant des dommages à la réputation et des amendes réglementaires. Dans certains cas, un attaquant peut obtenir une porte dérobée persistante dans les systèmes d'une organisation, entraînant une compromission à long terme qui peut passer inaperçue pendant une période prolongée.

## Exemples d'injection SQL

Il existe une grande variété de vulnérabilités, d'attaques et de techniques d'injection SQL, qui surviennent dans différentes situations. Voici quelques exemples courants d'injection SQL :

* Récupération des données cachées , où vous pouvez modifier une requête SQL pour renvoyer des résultats supplémentaires.
* Attaques UNION , où vous pouvez récupérer des données à partir de différentes tables de base de données.
* Examiner la base de données , où vous pouvez extraire des informations sur la version et la structure de la base de données.
* Injection SQL aveugle , où les résultats d'une requête que vous contrôlez ne sont pas renvoyés dans les réponses de l'application.

### Récupérer des données cachées

Considérez une application d'achat qui affiche des produits dans différentes catégories. Lorsque l'utilisateur clique sur la catégorie Cadeaux, son navigateur demande l'URL :
https://insecure-website.com/products?category=Gifts
Cela amène l'application à effectuer une requête SQL pour récupérer les détails des produits pertinents à partir de la base de données :

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Cette requête SQL demande à la base de données de retourner :

* tous les détails (\*)
* du tableau des produits
* où la catégorie est Cadeaux
* et libéré est 1.


La restriction released = 1est utilisée pour masquer les produits qui ne sont pas publiés. Pour les produits inédits, probablement released = 0.
L'application n'implémente aucune défense contre les attaques par injection SQL, donc un attaquant peut construire une attaque comme :
https://insecure-website.com/products?category=Gifts'--

Cela se traduit par la requête SQL :

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

L'élément clé ici est que la séquence à double tiret --est un indicateur de commentaire en SQL et signifie que le reste de la requête est interprété comme un commentaire. Cela supprime efficacement le reste de la requête, de sorte qu'elle n'inclut plus AND released = 1. Cela signifie que tous les produits sont affichés, y compris les produits non lancés.

Pour aller plus loin, un attaquant peut faire en sorte que l'application affiche tous les produits de n'importe quelle catégorie, y compris des catégories qu'il ne connaît pas :

https://insecure-website.com/products?category=Gifts'+OR+1=1--

Cela se traduit par la requête SQL :

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
La requête modifiée renverra tous les éléments dont la catégorie est Cadeaux ou 1 est égal à 1. Comme 1=1est toujours vrai, la requête renverra tous les éléments.



