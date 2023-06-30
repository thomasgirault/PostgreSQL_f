# SQLf : extension de postgreSQL au traitement de requêtes flexibles
==================================================================

Equipe IRISA/PILGRIM (http://www.irisa.fr/pilgrim/home_html)

Contributions scientifiques :
  Olivier Pivert (olivier.pivert AT irisa.fr)

Supervision du projet
  Gregory Smits (gregory.smits AT irisa.fr)

Auteur du code source :
  Thomas Girault (toma.girault AT gmail.com),

### Version de PostgreSQL supportée

Cette extension a été testée avec PostgreSQL 14 et GCC 11.3.0 sur Linux (Ubuntu 22.04).

### Installation

L'extension SQLf doit être installée sur un serveur PostgreSQL existant que l'on peut installer avec cette commande :

```
$ sudo apt install postgresql libpq-dev postgresql-server-dev-14 postgresql-plpython3-14 
```

Pour commencer une base de données doit être créée dans PostgreSQL.
L'extension fait usage de fonctions écrites dans les langages PL/PgSQL et PL/Python qui peuvent être installés de la manière suivante

```
$ createlang plpgsql <dbname>
$ createlang plpython3u <dbname>
```


Il d'abord faut cloner le projet :
```
$ git clone https://github.com/postgresqlf/PostgreSQL_f
```

La construction de l'extension nécessite ensuite une étape de compilation :

```
$ cd PostgreSQL_f
$ sudo make install
```

L'instruction 'CREATE EXTENSION sqlf;' permet de charger l'extension sqlf lors d'une connexion à une base de données :

```
$ su postgres
$ psql <dbname> -f 'CREATE EXTENSION sqlf;'
```

Cette commande va charger la librairie sqlf.so ainsi que l'ensemble du fichier sqlf--x.y.sql.
L'extension est alors opérationnelle et l'ensemble de ses fonctionnalités doivent être utilisables.


## Désinstallation

Pour supprimer l'extension et ses fonctions d'une base de données, on peut faitre appel au script de desinstallation sqlf_uninstall.sql :

```
$ psql -f sqlf_uninstall.sql <dbname>
```