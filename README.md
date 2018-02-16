#cassandra_tp

# 1: Approche relationnelle

> Nous allons étudier ici la création d’une base de données (appelée Keyspace), puis son interrogation. Cette première phase du TP consiste à créer la base comme si elle était relationnelle, et à effectuer des requêtes simples. Une fois les limites atteintes, nous utiliserons les spécificités de Cassandra pour aller plus loin.

### Création de la base de données

Avant d’interroger la base de données, il nous la créer. Pour commencer :

```sql
CREATE KEYSPACE IF NOT EXISTS resto_NY WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor': 1};
```
Nous créons ainsi une base de données resto_NY pour laquelle le facteur de réplication est mis à 1, ce qui suffit dans un cadre centralisé.

Sous `csqlsh`, vous pouvez maintenant sélectionner la base de données pour vos prochaines requêtes.

```sql
USE resto_NY;
```

L’équivalent existe dans une interface graphique bien entendu.

### Tables

Nous pouvons maintenant créer les tables (Column Family pour Cassandra) Restaurant et Inspection à partir du schéma suivant :

```sql
CREATE TABLE Restaurant (
   id INT, Name VARCHAR, borough VARCHAR, BuildingNum VARCHAR, Street VARCHAR,
   ZipCode INT, Phone text, CuisineType VARCHAR,
   PRIMARY KEY ( id )
 ) ;

 CREATE INDEX fk_Restaurant_cuisine ON Restaurant ( CuisineType ) ;

 CREATE TABLE Inspection (
   idRestaurant INT, InspectionDate date, ViolationCode VARCHAR,
   ViolationDescription VARCHAR, CriticalFlag VARCHAR, Score INT, GRADE VARCHAR,
   PRIMARY KEY ( idRestaurant, InspectionDate )
 ) ;

CREATE INDEX fk_Inspection_Restaurant ON Inspection ( Grade ) ;
```

Nous pouvons remarquer que chaque inspection est liée à un restaurant via l’identifiant de ce dernier.

Pour vérifier si les tables ont bien été créées (sous `cqlsh`).

```sql
DESC Restaurant;
DESC Inspection;
```

Nous pouvons voir le schéma des deux tables mais également des informations relatives au stockage dans la base Cassandra.

> *Import des données*
Maintenant, nous pouvons importer les fichiers CSV pour remplir les Column Family :

- Recuperer le fichier ‘restaurants.csv’ et ‘restaurants_inspections.csv’ dans le dossier `data`.
- Importer un fichier CSV via la console cqlsh

```sql
use resto_NY
COPY Restaurant (id, name, borough, buildingnum, street,
                    zipcode, phone, cuisinetype)
   FROM '/restaurants.csv' WITH DELIMITER=',';
COPY Inspection (idrestaurant, inspectiondate, violationcode,
                    violationdescription, criticalflag, score, grade)
   FROM '/restaurants_inspections.csv' WITH DELIMITER=',';
```

Pour vérifier le contenu des tables:

```sql
SELECT count(*) FROM Restaurant;
SELECT count(*) FROM Inspection;
```

### Interrogation

Les requêtes qui suivent sont à exprimer avec `CQL` (pour Cassandra Query Language) qui est fortement inspirée de SQL. Vous trouverez la syntaxe complète ici :

<https://cassandra.apache.org/doc/latest/cql/dml.html#select>).

### Requêtes CQL simples
- Pour la suite des exercices, exprimer en CQL les requêtes suivantes :
- Liste de tous les restaurants.
- Liste des Noms de restaurants.
- Nom et quartier (borough) du restaurant N° 41569764.
- Dates et grades des inspections de ce restaurant.
- Noms des restaurants de cuisine Française (French).
- Noms des restaurants situés dans BROOKLYN (attribut borough).
- Grades et scores donnés pour une inspection pour le restaurant n° 41569764 avec un score d’au moins 10.
- Grades (non nuls) des inspections dont le score est supérieur à 30.
- Nombre de lignes retournées par la requête précédente.
- Grades des inspections dont l’identifiant est compris entre 40 000 000 et 40 000 100.
Aide: Utiliser la fonction ‘token()‘.
- Compter le nombre de lignes retournées par la requête précédente.
Aide: Utiliser ‘COUNT(*)‘

### CQL Avancé

- Pour la requête ci-dessous faites en sorte qu’elle soit exécutable sans ALLOW FILTERING.
```sql
SELECT Name FROM Restaurant WHERE borough='BROOKLYN' ;
```
- Utilisons les deux indexes sur Restaurant (borough et cuisineType). Trouvez tous les noms de restaurants français de Brooklyn.
- Utiliser la commande TRACING ON avant la d’exécuter à nouveau la requête pour identifier quel index a été utilisé.
- On veut les noms des restaurants ayant au moins un grade ‘A’ dans leurs inspections. Est-ce possible en CQL?

