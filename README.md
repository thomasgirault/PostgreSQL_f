# SQLf : extension de postgreSQL au traitement de requêtes flexibles

Equipe IRISA/PILGRIM (http://www.irisa.fr/pilgrim/home_html)

Contributions scientifiques :
  Olivier Pivert (olivier.pivert AT irisa.fr)

Supervision du projet :
  Gregory Smits (gregory.smits AT irisa.fr)

Auteur du code source :
  Thomas Girault (toma.girault AT gmail.com),


## Introduction
Ce travail est réalisé dans le cadre d'une collaboration avec l'équipe IRISA/Pilgrim et l'association PostgreSQL-France.
L'objectif principal de ce travail est de permettre l'interrogation d'un système de gestion de bases de données relationnel (SGBDR) par des requêtes flexibles dont la particularité est de prendre en compte de conditions fondées sur les ensembles flous.
Cette extension introduit une collection de fonctions, opérateurs et casts qui permettent à PostgreSQL d'exécuter des requêtes flexibles.

## Installation
Testé sur Ubuntu 22.04
```
$ sudo apt install postgresql libpq-dev postgresql-server-dev-14 postgresql-plpython3-14 
$ git clone https://github.com/postgresqlf/PostgreSQL_f
$ cd PostgreSQL_f/
$ sudo make install
```

## Tests
```
$ psql mydatabase -f tests/annonces.dump
$ psql mydatabase -f tests/annonces.sql  
```

## Exemples d'utilisation
Pour cet exemple, nous employons une table de 200 enregistrements contenus dans le fichier tests/annonces.dump.
Ces enregistrements décrivent des véhicules en vente sur un site de petites annonces.
Chaque annonce est décrite par l'ensemble des champs suivants :
```
{idads, type, designation, price , mileage , year , make, name ,namecomp, model ,length ,width ,height ,nbdoors ,nbseats ,acceleration , consumption , co2emission , originalprice , iddb}
```
### 1 - Prédicats flous : définition et utilisation

Définition possible des prédicats "year is recent", "price is cheap", "milage is low".
SELECT create_predicate('recent', 1995., 2011., 0., 0.);
SELECT create_predicate('cheap', 0., 0., 5000.0, 20000.0);
SELECT create_predicate('low', 0., 0., 1000., 150000.);

Affectation d'un seuil ALPHA=0.2 pour la sélection de prédicats flous dont le score MU > ALPHA.

```
SELECT set_alpha(0.2);
```


Véhicules récents.

```
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE (year ~= 'recent')
ORDER BY mu DESC LIMIT 15;
```

Véhicules récents, bons marchés et ayant un kilométrage faible.

```
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE (price ~= 'very cheap') && (mileage ~= 'low') && (age ~= 'recent')
ORDER BY mu DESC LIMIT 15;
```

### 2 - Quantificateurs

Emploi du quantificateur OWA.
```
SELECT set_quantifier('owa');
```
Véhicules dont la plupart sont très bon marchés et dont le kilométrage est faible.
```
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE la_plupart(price ~= 'very cheap',mileage ~= 'low')
ORDER BY mu DESC LIMIT 15;
```

### 3 - Opérateurs graduels

Représentation d'un kilométrage (mileage) :
- type mileage + constructeur + opérateur CAST(INT AS mileage) ;
- distance entre deux kilométrages basée sur le coefficient de Tanimoto.
- opérateur graduel ~ pour mesurer une distance entre deux kilométrages.

```
DROP TYPE mileage cascade;
CREATE TYPE mileage AS (value FLOAT, mu FLOAT);
```
Constructeur
```
CREATE OR REPLACE FUNCTION mileage(INTEGER) RETURNS mileage AS
'SELECT ROW($1::float,get_mu())::mileage;'
LANGUAGE SQL IMMUTABLE;
```
CAST pour la conversion de INT vers mileage.
```
CREATE CAST (INTEGER AS mileage) WITH FUNCTION mileage(INTEGER);
```

Coefficient Tanimoto(a,b)=(a*b)/(a²+b²-a*b)
```
CREATE OR REPLACE FUNCTION distance(mileage, mileage) RETURNS FLOAT AS
'SELECT ($1.value * $2.value) / ($1.value^2 + $2.value^2 - $1.value * $2.value);'
LANGUAGE SQL IMMUTABLE;
```
opérateur graduel ~ pour le type mileage.
```
CREATE OPERATOR ~ (LEFTARG = mileage, RIGHTARG = mileage, PROCEDURE = distance);
```

Véhicules dont le kilométrage avoisine 98000 kms.
```
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE mileage::mileage ~ 98000::mileage
ORDER BY mu DESC LIMIT 15;
```

Véhicules dont le kilométrage avoisine 90000, 110000 ou 120000 kms.
```
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE mileage::mileage <~ ARRAY[90000, 110000, 120000]::mileage[]
ORDER BY mu DESC LIMIT 15;
```

##  Requêtes imbriquées

Véhicules dont la distance avoisine celle des véhicules très bons marchés.
````
SELECT DISTINCT make, name, model, price, mileage, year, get_mu() as mu
FROM annoncesauto
WHERE mileage::mileage <~~ ARRAY(SELECT mileage::mileage
                                  FROM annoncesauto
                                  WHERE (price ~= 'very cheap'))
ORDER BY mu DESC LIMIT 15;
```