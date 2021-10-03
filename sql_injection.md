# SQL Injection

## Qu'est-ce que l'injection SQL (SQLi) ?

![Injection SQL illustration 1](https://github.com/pluzeaux/bts-sio/blob/main/images/figure_1.png)

L'injection SQL est une vulnérabilité de sécurité Web qui permet à un attaquant d'interférer avec les requêtes qu'une application effectue sur sa base de données.

Il permet généralement à un attaquant de visualiser des données qu'il n'est normalement pas en mesure de récupérer. Cela peut inclure des données appartenant à d'autres utilisateurs ou toute autre donnée à laquelle l'application elle-même est en mesure d'accéder.

Dans de nombreux cas, un attaquant peut modifier ou supprimer ces données, provoquant des modifications persistantes du contenu ou du comportement de l'application.

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
```
https://insecure-website.com/products?category=Gifts
```

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
```
https://insecure-website.com/products?category=Gifts'--
```

Cela se traduit par la requête SQL :

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

L'élément clé ici est que la séquence à double tiret --est un indicateur de commentaire en SQL et signifie que le reste de la requête est interprété comme un commentaire. Cela supprime efficacement le reste de la requête, de sorte qu'elle n'inclut plus AND released = 1. Cela signifie que tous les produits sont affichés, y compris les produits non lancés.

Pour aller plus loin, un attaquant peut faire en sorte que l'application affiche tous les produits de n'importe quelle catégorie, y compris des catégories qu'il ne connaît pas :

```
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

Cela se traduit par la requête SQL :

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
La requête modifiée renverra tous les éléments dont la catégorie est Cadeaux ou 1 est égal à 1. Comme 1=1 est toujours vrai, la requête renverra tous les éléments.

## Subversion de la logique applicative

Considérez une application qui permet aux utilisateurs de se connecter avec un nom d'utilisateur et un mot de passe. Si un utilisateur soumet le nom d'utilisateur wieneret le mot de passe bluecheese, l'application vérifie les informations d'identification en effectuant la requête SQL suivante :

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

Si la requête renvoie les détails d'un utilisateur, la connexion est réussie. Sinon, il est rejeté.

Ici, un attaquant peut se connecter en tant qu'utilisateur sans mot de passe simplement en utilisant la séquence de commentaires SQL --pour supprimer la vérification du mot de passe de la WHEREclause de la requête. Par exemple, la soumission du nom d'utilisateur administrator'--et d'un mot de passe vide génère la requête suivante :

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

Cette requête renvoie l'utilisateur dont le nom d'utilisateur est administratoret connecte avec succès l'attaquant en tant qu'utilisateur.

## Récupérer des données à partir d'autres tables de base de données

Dans les cas où les résultats d'une requête SQL sont renvoyés dans les réponses de l'application, un attaquant peut exploiter une vulnérabilité d'injection SQL pour récupérer des données à partir d'autres tables de la base de données. Cela se fait à l'aide du UNIONmot - clé, qui vous permet d'exécuter une SELECTrequête supplémentaire et d'ajouter les résultats à la requête d'origine.

Par exemple, si une application exécute la requête suivante contenant l'entrée utilisateur « Cadeaux » :

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
```

alors un attaquant peut soumettre l'entrée :

```sql
' UNION SELECT username, password FROM users--
```

Cela obligera l'application à renvoyer tous les noms d'utilisateur et mots de passe ainsi que les noms et descriptions des produits.

## Examen de la base de données

Suite à l'identification initiale d'une vulnérabilité d'injection SQL, il est généralement utile d'obtenir des informations sur la base de données elle-même. Ces informations peuvent souvent ouvrir la voie à une exploitation ultérieure.

Vous pouvez interroger les détails de la version de la base de données. La façon dont cela est fait dépend du type de base de données, vous pouvez donc déduire le type de base de données à partir de la technique qui fonctionne. Par exemple, sur Oracle, vous pouvez exécuter :

```sql
SELECT * FROM v$version
```

Vous pouvez également déterminer quelles tables de base de données existent et quelles colonnes elles contiennent. Par exemple, sur la plupart des bases de données, vous pouvez exécuter la requête suivante pour répertorier les tables :

```sql
SELECT * FROM information_schema.tables
```

## Vulnérabilités d'injection SQL aveugle

De nombreuses instances d'injection SQL sont des vulnérabilités aveugles. Cela signifie que l'application ne renvoie pas les résultats de la requête SQL ou les détails des erreurs de base de données dans ses réponses. Des vulnérabilités aveugles peuvent toujours être exploitées pour accéder à des données non autorisées, mais les techniques impliquées sont généralement plus compliquées et difficiles à mettre en œuvre.

Selon la nature de la vulnérabilité et la base de données impliquée, les techniques suivantes peuvent être utilisées pour exploiter les vulnérabilités d'injection SQL aveugle :

* Vous pouvez modifier la logique de la requête pour déclencher une différence détectable dans la réponse de l'application en fonction de la véracité d'une seule condition. Cela peut impliquer l'injection d'une nouvelle condition dans une logique booléenne ou le déclenchement conditionnel d'une erreur telle qu'une division par zéro.
* Vous pouvez déclencher de manière conditionnelle un délai dans le traitement de la requête, ce qui vous permet de déduire la véracité de la condition en fonction du temps que met l'application pour répondre.
* Vous pouvez déclencher une interaction réseau hors bande à l'aide des techniques OAST . Cette technique est extrêmement puissante et fonctionne dans des situations où les autres techniques ne le font pas. Souvent, vous pouvez directement exfiltrer les données via le canal hors bande, par exemple en plaçant les données dans une recherche DNS pour un domaine que vous contrôlez.

## Comment détecter les vulnérabilités d'injection SQL

La majorité des vulnérabilités d'injection SQL peuvent être trouvées rapidement et de manière fiable à l'aide du scanner de vulnérabilités Web de Burp Suite .

L'injection SQL peut être détectée manuellement en utilisant un ensemble systématique de tests sur chaque point d'entrée de l'application. Cela implique généralement :

* Soumettre le caractère guillemet simple 'et rechercher des erreurs ou d'autres anomalies.
* Soumettre une syntaxe spécifique à SQL qui évalue la valeur de base (d'origine) du point d'entrée et une valeur différente, et rechercher des différences systématiques dans les réponses d'application résultantes.
* Soumettre des conditions booléennes telles que OR 1=1et OR 1=2, andrechercher des différences dans les réponses de l'application.
* Soumettre des charges utiles conçues pour déclencher des retards lorsqu'elles sont exécutées dans une requête SQL et rechercher des différences dans le temps nécessaire pour répondre.
* Soumettre des charges utiles OAST conçues pour déclencher une interaction réseau hors bande lorsqu'elles sont exécutées dans une requête SQL, et surveiller les interactions résultantes.

## Injection SQL dans différentes parties de la requête

La plupart des vulnérabilités d'injection SQL surviennent dans la WHEREclause d'une SELECTrequête. Ce type d'injection SQL est généralement bien compris par les testeurs expérimentés.

Mais les vulnérabilités d'injection SQL peuvent en principe se produire à n'importe quel endroit de la requête et au sein de différents types de requêtes. Les autres emplacements les plus courants où l'injection SQL se produit sont :

* Dans les UPDATEinstructions, dans les valeurs mises à jour ou la WHEREclause.
* Dans les INSERTinstructions, dans les valeurs insérées.
* Dans les SELECTinstructions, dans le nom de la table ou de la colonne.
* Dans les SELECTdéclarations, dans la ORDER BYclause.

## Injection SQL de second ordre

L'injection SQL de premier ordre se produit lorsque l'application prend l'entrée de l'utilisateur à partir d'une requête HTTP et, au cours du traitement de cette requête, incorpore l'entrée dans une requête SQL d'une manière dangereuse.

Dans l'injection SQL de second ordre (également appelée injection SQL stockée), l'application prend l'entrée de l'utilisateur à partir d'une requête HTTP et la stocke pour une utilisation future. Cela se fait généralement en plaçant l'entrée dans une base de données, mais aucune vulnérabilité ne survient au point où les données sont stockées. Plus tard, lors du traitement d'une requête HTTP différente, l'application récupère les données stockées et les intègre dans une requête SQL de manière non sécurisée.

L'injection SQL de second ordre se produit souvent dans des situations où les développeurs sont conscients des vulnérabilités de l'injection SQL et gèrent ainsi en toute sécurité le placement initial de l'entrée dans la base de données. Lorsque les données sont traitées ultérieurement, elles sont considérées comme sûres, car elles ont été précédemment placées dans la base de données en toute sécurité. À ce stade, les données sont traitées de manière dangereuse, car le développeur les considère à tort comme dignes de confiance.


## Facteurs spécifiques à la base de données

Certaines fonctionnalités de base du langage SQL sont implémentées de la même manière sur les plates-formes de bases de données courantes, et de nombreuses façons de détecter et d'exploiter les vulnérabilités d'injection SQL fonctionnent de manière identique sur différents types de bases de données.

Cependant, il existe également de nombreuses différences entre les bases de données communes. Cela signifie que certaines techniques de détection et d'exploitation de l'injection SQL fonctionnent différemment sur différentes plates-formes. Par exemple:

* Syntaxe pour la concaténation de chaînes.
* Commentaires.
* Requêtes groupées (ou empilées).
* API spécifiques à la plate-forme.
* Messages d'erreur.

## Comment empêcher l'injection SQL

La plupart des instances d'injection SQL peuvent être évitées en utilisant des requêtes paramétrées (également appelées instructions préparées) au lieu de la concaténation de chaînes dans la requête.

Le code suivant est vulnérable à l'injection SQL car l'entrée utilisateur est concaténée directement dans la requête :

```sql
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
```

```
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

Ce code peut être facilement réécrit d'une manière qui empêche l'entrée de l'utilisateur d'interférer avec la structure de la requête :

```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

Les requêtes paramétrées peuvent être utilisées pour toute situation où une entrée non fiable apparaît sous forme de données dans la requête, y compris la WHEREclause et les valeurs d'une instruction INSERTou UPDATE. Ils ne peuvent pas être utilisés pour gérer des entrées non fiables dans d'autres parties de la requête, telles que les noms de table ou de colonne, ou la ORDER BYclause. La fonctionnalité d'application qui place des données non fiables dans ces parties de la requête devra adopter une approche différente, telle que la mise en liste blanche des valeurs d'entrée autorisées, ou l'utilisation d'une logique différente pour fournir le comportement requis.

Pour qu'une requête paramétrée empêche efficacement l'injection SQL, la chaîne utilisée dans la requête doit toujours être une constante codée en dur et ne doit jamais contenir de données variables de quelque origine que ce soit. Ne soyez pas tenté de décider au cas par cas si un élément de données est fiable et continuez à utiliser la concaténation de chaînes dans la requête pour les cas considérés comme sûrs. Il est trop facile de faire des erreurs sur l'origine possible des données, ou que des modifications dans d'autres codes violent les hypothèses sur les données qui sont entachées.


