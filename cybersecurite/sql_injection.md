# SQL Injection

## Qu'est-ce que l'injection SQL (SQLi) ?

![Injection SQL illustration 1](https://github.com/pluzeaux/bts-sio/blob/main/images/figure_1.png)

L'injection *SQL* est une vulnérabilité de sécurité Web qui permet à un attaquant d'interférer avec les requêtes qu'une application effectue sur sa base de données.  

Il permet généralement à un attaquant de visualiser des données qu'il n'est normalement pas en mesure de récupérer. Cela peut inclure des données appartenant à d'autres utilisateurs ou toute autre donnée à laquelle l'application elle-même est en mesure d'accéder.

Dans de nombreux cas, un attaquant peut modifier ou supprimer ces données, provoquant des modifications persistantes du contenu ou du comportement de l'application.

Dans certaines situations, un attaquant peut intensifier une attaque par injection SQL pour compromettre le serveur sous-jacent ou une autre infrastructure principale, ou effectuer une attaque par déni de service.

## Quel est l'impact d'une attaque par injection SQL réussie ?

Une attaque par injection *SQL* réussie peut entraîner un accès non autorisé à des données sensibles, telles que des mots de passe, des détails de carte de crédit ou des informations personnelles sur l'utilisateur. De nombreuses violations de données très médiatisées ces dernières années ont été le résultat d'attaques par injection *SQL*, entraînant des dommages à la réputation et des amendes réglementaires.  
Dans certains cas, un attaquant peut obtenir une porte dérobée persistante dans les systèmes d'une organisation, entraînant une compromission à long terme qui peut passer inaperçue pendant une période prolongée.  

## Exemples d'injection *SQL*

Il existe une grande variété de vulnérabilités, d'attaques et de techniques d'injection SQL, qui surviennent dans différentes situations. Voici quelques exemples courants d'injection *SQL* :

* Récupération des données cachées , où vous pouvez modifier une requête SQL pour renvoyer des résultats supplémentaires.
* Attaques `UNION` , où vous pouvez récupérer des données à partir de différentes tables de base de données.
* Examiner la base de données , où vous pouvez extraire des informations sur la version et la structure de la base de données.
* Injection *SQL* aveugle , où les résultats d'une requête que vous contrôlez ne sont pas renvoyés dans les réponses de l'application.

### Récupérer des données cachées

Considérez une application d'achat qui affiche des produits dans différentes catégories. Lorsque l'utilisateur clique sur la catégorie Cadeaux, son navigateur demande l'URL :
```
https://insecure-website.com/products?category=Gifts
```

Cela amène l'application à effectuer une requête *SQL* pour récupérer les détails des produits pertinents à partir de la base de données :

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
Cette requête *SQL* demande à la base de données de retourner :
* tous les détails (\*)
* de la table `produits`
* où la `category` est `Gifts`
* et `released` est 1.


La restriction `released = 1` est utilisée pour masquer les produits qui ne sont pas publiés. Pour les produits inédits, probablement `released = 0`.
L'application n'implémente aucune défense contre les attaques par injection *SQL*, donc un attaquant peut construire une attaque comme :

```
https://insecure-website.com/products?category=Gifts'--
```

Cela se traduit par la requête SQL :

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

L'élément clé ici est que la séquence à double tiret `--` est un indicateur de commentaire en *SQL* et signifie que le reste de la requête est interprété comme un commentaire. Cela supprime efficacement le reste de la requête, de sorte qu'elle n'inclut plus `AND released = 1`. Cela signifie que tous les produits sont affichés, y compris les produits non lancés.

Pour aller plus loin, un attaquant peut faire en sorte que l'application affiche tous les produits de n'importe quelle catégorie, y compris des catégories qu'il ne connaît pas :

```
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

Cela se traduit par la requête *SQL* :

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
La requête modifiée renverra tous les éléments dont la `category` est `Gifts` ou `1` est égal à `1`. Comme `1=1` est toujours vrai, la requête renverra tous les éléments.

## Subversion de la logique applicative

Considérez une application qui permet aux utilisateurs de se connecter avec un nom d'utilisateur et un mot de passe.  
Si un utilisateur soumet le nom d'utilisateur `wieneret` le mot de passe `bluecheese`, l'application vérifie les informations d'identification en effectuant la requête *SQL* suivante :

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

Si la requête renvoie les détails d'un utilisateur, la connexion est réussie. Sinon, il est rejeté.
Ici, un attaquant peut se connecter en tant qu'utilisateur sans mot de passe simplement en utilisant la séquence de commentaires *SQL* `--` pour supprimer la vérification du mot de passe de la clause `WHERE` de la requête. Par exemple, la soumission du nom d'utilisateur `administrator'--` et d'un mot de passe vide génère la requête suivante :

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

Cette requête renvoie l'utilisateur dont le nom est `administrator` et connecte avec succès l'attaquant en tant qu'utilisateur.

## Récupérer des données à partir d'autres tables de base de données

Dans les cas où les résultats d'une requête *SQL* sont renvoyés dans les réponses de l'application, un attaquant peut exploiter une vulnérabilité d'injection *SQL* pour récupérer des données à partir d'autres tables de la base de données.  
Cela se fait à l'aide du `UNION` mot-clé, qui vous permet d'exécuter un `SELECT`  supplémentaire et d'ajouter les résultats à la requête d'origine.


Par exemple, si une application exécute la requête suivante contenant l'entrée utilisateur « Gifts » :

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
```

alors un attaquant peut soumettre l'entrée :

```sql
' UNION SELECT username, password FROM users--
```

Cela obligera l'application à renvoyer tous les noms d'utilisateur et mots de passe ainsi que les noms et descriptions des produits.

### Attaques UNION par injection SQL

Lorsqu'une application est vulnérable à l'injection *SQL* et que les résultats de la requête sont renvoyés dans les réponses de l'application, le mot-clé `UNION` peut être utilisé pour récupérer des données à partir d'autres tables de la base de données. Cela entraîne une attaque `UNION` par injection *SQL*.

Le mot-clé *UNION* vous permet d'exécuter une ou plusieurs requêtes *SELECT* supplémentaires et d'ajouter les résultats à la requête d'origine.
Par exemple:

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

Cette requête *SQL* renverra un seul jeu de résultats avec deux colonnes, contenant les valeurs des colonnes `a` et `b` dans `table1` et des colonnes `c` et `d` dans `table2`.

Pour qu'une requête `UNION` fonctionne, deux exigences clés doivent être remplies :

Les requêtes individuelles doivent renvoyer le même nombre de colonnes.
Les types de données dans chaque colonne doivent être compatibles entre les requêtes individuelles.
Pour effectuer une attaque `UNION` par injection *SQL*, vous devez vous assurer que votre attaque répond à ces deux exigences.

Il s'agit généralement de déterminer :
* Combien de colonnes sont renvoyées à partir de la requête d'origine ?
* Quelles colonnes renvoyées par la requête d'origine sont d'un type de données approprié pour contenir les résultats de la requête injectée ?

#### Détermination du nombre de colonnes requises dans une attaque UNION par injection SQL

Lors de l'exécution d'une attaque `UNION` par injection *SQL*, il existe deux méthodes efficaces pour déterminer le nombre de colonnes renvoyées par la requête d'origine.

La première méthode consiste à injecter une série de `ORDER BY` clauses et à incrémenter l'index de colonne spécifié jusqu'à ce qu'une erreur se produise. Par exemple, en supposant que le point d'injection est une chaîne entre guillemets dans la `WHERE` clause de la requête d'origine, vous devez soumettre :

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

Cette série d'instructions modifie la requête d'origine pour ordonner les résultats par différentes colonnes dans l'ensemble de résultats. La colonne d'un `ORDER BY` clause peut être spécifiée par son index, vous n'avez donc pas besoin à connaître le nom des colonnes. Lorsque l'index de colonne spécifié dépasse le nombre de colonnes réelles dans le jeu de résultats, la base de données renvoie une erreur, telle que :

```sql
The `ORDER BY` position number 3 is out of range of the number of items in the select list.
```
L'application peut en fait renvoyer l'erreur de base de données dans sa réponse *HTTP*, ou elle peut renvoyer une erreur générique, ou simplement ne renvoyer aucun résultat. À condition que vous puissiez détecter une différence dans la réponse de l'application, vous pouvez déduire le nombre de colonnes renvoyées à partir de la requête.

La deuxième méthode consiste à soumettre une série de `UNION SELECT` spécifiant un nombre différent de valeurs nulles :

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

Si le nombre de valeurs `NULL` ne correspond pas au nombre de colonnes, la base de données renvoie une erreur, telle que :

```sql
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```

Encore une fois, l'application peut en fait renvoyer ce message d'erreur, ou peut simplement renvoyer une erreur générique ou aucun résultat.  
Lorsque le nombre de valeurs `NULL` correspond au nombre de colonnes, la base de données renvoie une ligne supplémentaire dans le jeu de résultats, contenant des valeurs `NULL` dans chaque colonne.  
L'effet sur la réponse *HTTP* résultante dépend du code de l'application. Si vous avez de la chance, vous verrez du contenu supplémentaire dans la réponse, comme une ligne supplémentaire sur un tableau *HTML*.  
Sinon, les valeurs nulles pourraient déclencher une erreur différente, telle qu'une exception de type  `NullPointerException`.  
Dans le pire des cas, la réponse peut être impossible à distinguer de celle qui est provoquée par un nombre incorrect de valeurs `NULL`, rendant cette méthode de détermination du nombre de colonnes inefficace.

!!! note
    La raison de l'utilisation `NULL` des valeurs renvoyées par la requête `SELECT` injectée est que les types de données dans chaque colonne doivent être compatibles entre les requêtes d'origine et injectées.  
    Étant donné que `NULL` est convertible en tous les types de données couramment utilisés, l'utilisation de `NULL` maximise les chances que l'injectione réussisse lorsque le nombre de colonnes est correct.  
    Sur *Oracle*, chaque requête `SELECT` doit utiliser le mot-clé `FROM` et spécifier une table valide. Il existe une table intégrée sur *Oracle* appelée `DUAL` qui peut être utilisée à cette fin. 

    Ainsi, les requêtes injectées sur *Oracle* devraient ressembler à :  
    `' UNION SELECT NULL FROM DUAL--`. 

    Les injections décrites utilisent la séquence `--` de commentaires à double tiret pour commenter le reste de la requête d'origine après le point d'injection.  
    Sur *MySQL*, la séquence de double tiret doit être suivie d'un espace. Alternativement, le caractère dièse `#` peut être utilisé pour identifier un commentaire.

Recherche de colonnes avec un type de données utile dans une attaque `UNION` par injection *SQL*

La raison d'effectuer une attaque `UNION` par injection *SQL* est de pouvoir récupérer les résultats d'une requête injectée. En règle générale, les données intéressantes que vous souhaitez récupérer seront sous forme de chaîne, vous devez donc rechercher une ou plusieurs colonnes dans les résultats de la requête d'origine dont le type de données est ou est compatible avec les données de chaîne.

Après avoir déjà déterminé le nombre de colonnes requises, vous pouvez sonder chaque colonne pour tester si elle peut contenir des données de chaîne en soumettant une série de `UNION SELECT` qui placent une valeur de chaîne dans chaque colonne à tour de rôle. Par exemple, si la requête renvoie quatre colonnes, vous devez soumettre :

```
' UNION SELECT 'a',  NULL, NULL, NULL--
' UNION SELECT NULL, 'a',  NULL, NULL--
' UNION SELECT NULL, NULL, 'a',  NULL--
' UNION SELECT NULL, NULL, NULL, 'a' --
```

Si le type de données d'une colonne n'est pas compatible avec les données de chaîne, la requête injectée provoquera une erreur de base de données, telle que :

```sql
Conversion failed when converting the varchar value 'a' to data type int.
```

Si aucune erreur ne se produit et que la réponse de l'application contient du contenu supplémentaire, y compris la valeur de chaîne injectée, la colonne appropriée convient pour récupérer les données de chaîne.

#### Utiliser une attaque UNION par injection SQL pour récupérer des données intéressantes

Lorsque vous avez déterminé le nombre de colonnes renvoyées par la requête d'origine et trouvé quelles colonnes peuvent contenir des données de chaîne, vous êtes en mesure de récupérer des données intéressantes.

Supposer que:

* La requête d'origine renvoie deux colonnes, qui peuvent toutes deux contenir des données de chaîne.
* Le point d'injection est une chaîne entre guillemets dans la WHEREclause.
* La base de données contient une table appelée usersavec les colonnes usernameet password.

Dans cette situation, vous pouvez récupérer le contenu de la userstable en soumettant l'entrée :

```sql
' UNION SELECT username, password FROM users--
```

Bien sûr, l'information cruciale nécessaire pour effectuer cette attaque est qu'il existe une table appelée `users` avec deux colonnes appelées username et `password`.  
Sans ces informations, vous seriez obligé de deviner les noms des tables et des colonnes.  
En fait, toutes les bases de données modernes offrent des moyens d'examiner la structure de la base de données, afin de déterminer quelles tables et colonnes elle contient.


#### Examen de la base de données dans les attaques par injection SQL

Lors de l'exploitation des vulnérabilités d' injection *SQL* , il est souvent nécessaire de collecter des informations sur la base de données elle-même. Cela inclut le type et la version du logiciel de base de données, ainsi que le contenu de la base de données en termes de tables et de colonnes qu'elle contient.

##### Interrogation du type et de la version de la base de données

Différentes bases de données offrent différentes manières d'interroger leur version. Vous devez souvent essayer différentes requêtes pour en trouver une qui fonctionne, vous permettant de déterminer à la fois le type et la version du logiciel de base de données.

Les requêtes pour déterminer la version de la base de données pour certains types de base de données courants sont les suivantes :

| Type de base de données  | Mettre en doute            
|--------------------------|---------------------------
| Microsoft, MySQL	       | `SELECT @@version`     
| Oracle	               | `SELECT * FROM v$version`
| PostgreSQL	           | `SELECT version()`

Par exemple, vous pouvez utiliser une UNIONattaque avec l'entrée suivante :

```sql
' UNION SELECT @@version--
```

Cela peut renvoyer une sortie comme celle-ci, confirmant que la base de données est Microsoft *SQL Server* et la version utilisée :

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```


##### Lister le contenu de la base de données

La plupart des types de bases de données (à l'exception notable d'*Oracle*) ont un ensemble de vues appelé schéma d'informations qui fournissent des informations sur la base de données.

Vous pouvez interroger information_schema.tablespour répertorier les tables de la base de données :

```sql
SELECT * FROM information_schema.tables
```

Cela renvoie une sortie comme suit :

```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE
=====================================================
MyDatabase dbo Products BASE TABLE
MyDatabase dbo Users BASE TABLE
MyDatabase dbo Feedback BASE TABLE
```

Cette sortie indique qu'il existe trois tables, appelées `Products`, `Users`, et `Feedback`.

Vous pouvez ensuite interroger `information_schema.columns` pour répertorier les colonnes dans des tables individuelles :

```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

Cela renvoie une sortie comme suit :

```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE
=================================================================
MyDatabase dbo Users UserId int
MyDatabase dbo Users Username varchar
MyDatabase dbo Users Password varchar
```

Cette sortie affiche les colonnes de la table spécifiée et le type de données de chaque colonne.

Équivalent au schéma d'information sur *Oracle*
Sur *Oracle*, vous pouvez obtenir les mêmes informations avec des requêtes légèrement différentes.

Vous pouvez lister les tables en interrogeant all_tables:
```sql
SELECT * FROM all_tables
```

Et vous pouvez lister les colonnes en interrogeant `all_tab_columns`:

```sql
SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
```

#### Récupérer plusieurs valeurs dans une seule colonne
Dans l'exemple précédent, supposons plutôt que la requête ne renvoie qu'une seule colonne.

Vous pouvez facilement récupérer plusieurs valeurs ensemble dans cette seule colonne en concaténant les valeurs ensemble, en incluant idéalement un séparateur approprié pour vous permettre de distinguer les valeurs combinées. Par exemple, sur *Oracle*, vous pouvez soumettre l'entrée :

```sql
' UNION SELECT username || '~' || password FROM users--
```

Cela utilise la séquence à double tube ||qui est un opérateur de concaténation de chaînes sur Oracle. La requête injectée concatène les valeurs des champs usernameet password, séparées par le ~caractère.

Les résultats de la requête vous permettront de lire tous les noms d'utilisateur et mots de passe, par exemple :

```
...
administrator~s3cure
wiener~peter
carlos~montoya
...
```

Notez que différentes bases de données utilisent une syntaxe différente pour effectuer la concaténation de chaînes. Pour plus de détails, consultez l' aide-mémoire sur l'injection SQL .



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

Out-of-band application security testing (OAST) utilise des serveurs externes pour voir des vulnérabilités autrement invisibles. Il a été introduit pour améliorer encore le modèle DAST (dynamic application security testing). 

## Comment détecter les vulnérabilités d'injection SQL

La majorité des vulnérabilités d'injection SQL peuvent être trouvées rapidement et de manière fiable à l'aide du scanner de vulnérabilités Web de Burp Suite .

L'injection SQL peut être détectée manuellement en utilisant un ensemble systématique de tests sur chaque point d'entrée de l'application. Cela implique généralement :

* Soumettre le caractère guillemet simple `'` et rechercher des erreurs ou d'autres anomalies.
* Soumettre une syntaxe spécifique à *SQL* qui évalue la valeur de base (d'origine) du point d'entrée et une valeur différente, et rechercher des différences systématiques dans les réponses d'application résultantes.
* Soumettre des conditions booléennes telles que `OR 1=1` et `OR 1=2`, et rechercher des différences dans les réponses de l'application.
* Soumettre des charges utiles conçues pour déclencher des retards lorsqu'elles sont exécutées dans une requête *SQL* et rechercher des différences dans le temps nécessaire pour répondre.
* Soumettre des charges utiles *OAST* conçues pour déclencher une interaction réseau hors bande lorsqu'elles sont exécutées dans une requête *SQL*, et surveiller les interactions résultantes.


## Injection SQL dans différentes parties de la requête

La plupart des vulnérabilités d'injection SQL surviennent dans la WHEREclause d'une SELECTrequête. Ce type d'injection SQL est généralement bien compris par les testeurs expérimentés.

Mais les vulnérabilités d'injection SQL peuvent en principe se produire à n'importe quel endroit de la requête et au sein de différents types de requêtes. Les autres emplacements les plus courants où l'injection SQL se produit sont :

* Dans les instructions *UPDATE*, dans les valeurs mises à jour ou la clause WHERE.
* Dans les instructions *INSERT*, dans les valeurs insérées.
* Dans les instructions *SELECT*, dans le nom de la table ou de la colonne.
* Dans les déclarations *SELECT*, dans la clause *ORDER BY*.

## Injection SQL de second ordre

L'injection SQL de premier ordre se produit lorsque l'application prend l'entrée de l'utilisateur à partir d'une requête HTTP et, au cours du traitement de cette requête, incorpore l'entrée dans une requête SQL d'une manière dangereuse.

Dans l'injection SQL de second ordre (également appelée injection SQL stockée), l'application prend l'entrée de l'utilisateur à partir d'une requête HTTP et la stocke pour une utilisation future. Cela se fait généralement en plaçant l'entrée dans une base de données, mais aucune vulnérabilité ne survient au point où les données sont stockées. Plus tard, lors du traitement d'une requête HTTP différente, l'application récupère les données stockées et les intègre dans une requête SQL de manière non sécurisée.

![Injection SQL illustration 1](https://github.com/pluzeaux/bts-sio/blob/main/images/figure_2.png)

L'injection SQL de second ordre se produit souvent dans des situations où les développeurs sont conscients des vulnérabilités de l'injection SQL et gèrent ainsi en toute sécurité le placement initial de l'entrée dans la base de données. Lorsque les données sont traitées ultérieurement, elles sont considérées comme sûres, car elles ont été précédemment placées dans la base de données en toute sécurité. À ce stade, les données sont traitées de manière dangereuse, car le développeur les considère à tort comme dignes de confiance.


## Facteurs spécifiques à la base de données

Certaines fonctionnalités de base du langage *SQL* sont implémentées de la même manière sur les plates-formes de bases de données courantes, et de nombreuses façons de détecter et d'exploiter les vulnérabilités d'injection SQL fonctionnent de manière identique sur différents types de bases de données.

Cependant, il existe également de nombreuses différences entre les bases de données communes. Cela signifie que certaines techniques de détection et d'exploitation de l'injection SQL fonctionnent différemment sur différentes plates-formes. Par exemple:

* Syntaxe pour la concaténation de chaînes.
* Commentaires.
* Requêtes groupées (ou empilées).
* API spécifiques à la plate-forme.
* Messages d'erreur.

## Comment empêcher l'injection SQL

La plupart des instances d'injection *SQL* peuvent être évitées en utilisant des requêtes paramétrées (également appelées instructions préparées) au lieu de la concaténation de chaînes dans la requête.

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

Les requêtes paramétrées peuvent être utilisées pour toute situation où une entrée non fiable apparaît sous forme de données dans la requête, y compris la `WHERE` clause et les valeurs d'une instruction `INSERT` ou `UPDATE`. Ils ne peuvent pas être utilisés pour gérer des entrées non fiables dans d'autres parties de la requête, telles que les noms de table ou de colonne, ou la `ORDER BY` clause. La fonctionnalité d'application qui place des données non fiables dans ces parties de la requête devra adopter une approche différente, telle que la mise en liste blanche des valeurs d'entrée autorisées, ou l'utilisation d'une logique différente pour fournir le comportement requis.

Pour qu'une requête paramétrée empêche efficacement l'injection *SQL*, la chaîne utilisée dans la requête doit toujours être une constante codée en dur et ne doit jamais contenir de données variables de quelque origine que ce soit. Ne soyez pas tenté de décider au cas par cas si un élément de données est fiable et continuez à utiliser la concaténation de chaînes dans la requête pour les cas considérés comme sûrs. Il est trop facile de faire des erreurs sur l'origine possible des données, ou que des modifications dans d'autres codes violent les hypothèses sur les données qui sont entachées.


