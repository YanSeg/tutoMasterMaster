# Tuto Master_Master 

https://mariadb.com/kb/en/setting-up-replication/

## Avoir installer docker et docker compose


## Au besoin, pour éviter de vous perdre, repartez de 0 !

  - Supprimer ces images docker :

```
docker images -q
```
```
docker rmi $(docker images -q)
```

 - Supprimer ces volumes docker : 

```
docker volume ls -q
```
```
docker volume rm $(docker volume ls -q)
```
- Avec cela vous êtes certain de gagner en place sur votre ordi et de repartir à chaque fois d'une configuration vierge.

## 1 / Lancer les 3 conatiners avec le docker-compose.yml

- Se rendre dans le dossier ou il y a le docker compose (cd tutoMasterMaster)

```
docker compose up 
```
(si votre docker-compose.yml porte un autre nom : docker compose -f name.yml up )

- Vérifier la bonne création des 3 containers
``` 
yans@yans-IdeaPad-3-15IML05:~$ docker ps 
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                                                                                  NAMES
3556b2ddae97   yanseg/worms   "/usr/local/bin/entr…"   10 seconds ago   Up 8 seconds   0.0.0.0:8081->80/tcp, :::8081->80/tcp, 0.0.0.0:19998->19999/tcp, :::19998->19999/tcp   WORDPRESS
0dbf36707800   mariadb        "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   3306/tcp                                                                               SQL-MASTER
fb7b2a96514f   mariadb        "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   3306/tcp                                                                               SQL-MASTER01

```


## 2 / Récupérer les IP des deux master

```
./ipContainer.sh SQL-MASTER

L'adresse IP du conteneur 'SQL-MASTER' est : 172.22.0.3
```
```
./ipContainer.sh SQL-MASTER01

L'adresse IP du conteneur 'SQL-MASTER01' est : 172.22.0.2

```

## 3 / Se rendre dans le MASTER 1

- Récupérer l adresse IP du container SQL-MASTER
``` 
yans@yans-IdeaPad-3-15IML05:~/Bureau/tuto_Master-Master$ ./ipContainer.sh SQL-MASTER
L'adresse IP du conteneur 'SQL-MASTER' est : 172.22.0.3
```

- Se rendre dans le docker master (SQL-MASTER) :
```
docker exec -it SQL-MASTER bash 
```

- Se connecter à mariadb
```
 mariadb -u root -psomewordpress
```
- Puis suivre ces étapes :
```
CREATE USER 'master_user'@'%' IDENTIFIED BY 'master';
```

```
GRANT REPLICATION SLAVE ON *.* TO 'master_user'@'%';
```
```
SHOW MASTER STATUS;

MariaDB [(none)]> SHOW MASTER STATUS;
+-------------------+----------+--------------+------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| mysqld-bin.000002 |      672 |              |                  |
+-------------------+----------+--------------+------------------+
1 row in set (0.000 sec)


```

_______________________________________________________________________
- Noter cela pour la suite 
```

CHANGE MASTER TO
  MASTER_HOST='172.22.0.3',
  MASTER_USER='master_user',
  MASTER_PASSWORD='master',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysqld-bin.000002',
  MASTER_LOG_POS=672,
  MASTER_CONNECT_RETRY=10;

```
______________________________________________________________________
 
 - Gardez le terminal votre container SQL-MASTER ouvert, vous allez bientôt relancer quelques requêtes SQL. 


## 4/ Se rendre dans le MASTER 2 


- Ouvrir un nouveau terminal et se rendre dans le docker master (SQL-MASTER01) :

```
docker exec -it SQL-MASTER01 bash 
```


- Se connecter à mariadb
```
 mariadb -u root -psomewordpress

```
- Puis suivre ces étapes :
```
CREATE USER 'mastro_user'@'%' IDENTIFIED BY 'maestro';
```

```
GRANT REPLICATION SLAVE ON *.* TO 'maestro_user'@'%';
```

```
show master status;
```
```
MariaDB [(none)]> show master status;
+-------------------+----------+--------------+------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| mysqld-bin.000002 |      675 |              |                  |
+-------------------+----------+--------------+------------------+
1 row in set (0.000 sec)
```
_______________________________________________________________________
- Notez cela pour la suite :
```
CHANGE MASTER TO
  MASTER_HOST='172.22.0.2',
  MASTER_USER='maestro_user',
  MASTER_PASSWORD='maestro',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysqld-bin.000002',
  MASTER_LOG_POS=675,
  MASTER_CONNECT_RETRY=10;
```
________________________________________________________________

- Puis continuer par :
```
STOP SLAVE;
```
```
CHANGE MASTER TO
  MASTER_HOST='172.22.0.2',
  MASTER_USER='master_user',
  MASTER_PASSWORD='master',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysqld-bin.000002',
  MASTER_LOG_POS=672,
  MASTER_CONNECT_RETRY=10;

```
```
start slave;
```




## 5/ Se rendre  à nouveau dans le MASTER 1 

- Continuer par :

```
stop slave;
```
```
CHANGE MASTER TO
  MASTER_HOST='172.22.0.2',
  MASTER_USER='maestro_user',
  MASTER_PASSWORD='maestro',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='mysqld-bin.000002',
  MASTER_LOG_POS=675,
  MASTER_CONNECT_RETRY=10;
```
```
start slave;
```

## 6 / Bonnes fêtes de fin d'année.
- Testez le bon fonctionnement comme bon vous semble. 

Sur n'importe lequel des containers 
```
create database Tumedois10e
```
Puis voir que la bdd est bien créée sur l'auttre :
```
show databases;
```




