# Réplication SQL

## Notes

Dans le cadre de ce document, nous effectuerons un test de réplication en utilisateur deux Raspberry PI.
Le même exercice peut être réalisé en utilisant deux containers, deux machines virtuelles ou, à moindre mesure, sur une même machine.

La documentation quant à la réplication pour MariaDB est [très fournie](https://mariadb.com/docs/server/ha-and-performance/standard-replication) et vous pouvez simplement vous lancer de vous-même.
La suite de ce document ne sert qu'à donner des pistes pour faciliter la progression et les tests sur le sujet de la réplication.

## Pré-requis

L'installation de MySQL/MariaDB sur les deux postes doit être réalisée. Il est fortement conseillé de conserver le même type de système à jour entre les postes.
Un poste sera élu "master" et un autre "slave".

> [!Note]
> Pour des raisons logiques, le [nommage de ces mécanismes est remis en question depuis quelques années](https://en.wikipedia.org/wiki/Master%E2%80%93slave_(technology)#Controversy).
> Pour des raisons de praticités vis-à-vis des documentations existentes, ce document utilisera le vocable master/slave, mais des vocables main/replica ou primary/replica peut être aussi adopté sans problèmes.

## Objectif

La réplication est un mécanisme automatisé de réplication de données entre un serveur "master" et un ou plusieurs serveurs "slave".
Le serveur "master" est le garant des données, et lui seul peut effectuer des modifications de données et des envois aux slaves, tandis que les slaves ne peuvent effectuer de modification de données par eux-mêmes ou envoyer des données.

## Activité

Pour cette activité, nous allons répliquer une base de test, nommé facétieusement `demorepl`.

```sql
CREATE DATABASE demorepl;
USE demorepl;
CREATE TABLE users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100));
INSERT INTO users (name) VALUES ('Alice'), ('Bob');
```

### Activation des binary logs

La première étape pour ajouter la réplication est d'activer le mécanisme de binary log, qui est un journal de modifications sous un format spécial, permettant de "rejouer" les opéarations effectuées.

L'ajout du binary log sur une base de données particulière passe par la [modification de la configuration du serveur](https://mariadb.com/docs/server/ha-and-performance/standard-replication/setting-up-replication) :

```ini
[mariadb]
log-bin
server_id=1
log-basename=master1
binlog-format=mixed
binlog_do_db = demorepl
```

> [!Note]
> La directive binlog_do_db est un [filtre permettant de n'activer la réplication que sur une liste précise de bases de données](https://mariadb.com/docs/server/ha-and-performance/standard-replication/replication-filters).

### S'assurer des permissions d'un utilisateur dédié

Il faut ensuite s'assurer des permissions pour un utilisateur qui se chargera de la réplication. Pour l'exemple, nous créons un utilisateur dédié.

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

> [!Tip]
> La commande FLUSH PRIVILEGES permet de s'assurer que les privilèges associés soient rechargés sans devoir relancer une connexion.
> Cette commande est en réalité un reliquat, et n'est pas nécessaire si vous passez par GRANT et n'utilisez pas en direct les tables systèmes.

### Vérification réseau

Vérifiez les directives:
- `skip-networking` , qui empêche l'usage du réseau, et donc bloque la réplication hors d'un même hôte
- `bind-address` , qui limite les adresses pouvant interagir avec le serveur (pour résumer)

### Vérification coté master

Vérifions l'état coté master :

```sql
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

La première commande permet de verrouiller les tables en lecture.

### Configuration slave

Sur le slave, ajoutez simplement un server-id différent.

### Copie manuelle

Copions manuellement notre master vers notre slave. 

Il existe un [grand nombre de moyens de copier et d'importer une base sous MariaDB](https://mariadb.com/docs/server/clients-and-utilities/backup-restore-and-import-clients), nous allons simplement dump puis importer la base après transfert.

```
mysqldump -u root -p demorepl > dump-demorepl.sql
```

Puis, on transfert le fichier vers le serveur slave (note : cela signifie que vous avez configuré votre connexion SSH entre les deux serveurs !) :

```
scp dump-demorepl.sql <user>@<ip>:/tmp
```

Enfin, on va lancer l'import sur le serveur slave (donc connectez-vous bien sur le serveur slave !) :

```
mysql -u root -p < /tmp/dump-demorepl.sql
```

### Suite de la configuration du slave

Maintenant, nous pouvons finir de configurer le slave :

```sql
CHANGE MASTER TO
  MASTER_HOST='master.domain.com',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='bigs3cret',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='master1-bin.000096',
  MASTER_LOG_POS=568,
  MASTER_CONNECT_RETRY=10;
```

> [!Warning]
> Beaucoup de ces éléments doivent être modifiés selon votre cas. Ces options sont décrites dans la [documentation de CHANGE MASTER TO](https://mariadb.com/docs/server/reference/sql-statements/administrative-sql-statements/replication-statements/change-master-to#syntax).
> Basez-vous sur les différentes options décrites auparavant pour trouver ce qui correspond à notre cas !

Enfin, ne reste qu'à démarrer la réplication : `START SLAVE`.

D'après vous, est-ce que la réplication fonctionne par :
- "pull", ce qui signifie que les slaves vont "aller chercher" les dernières modifications depuis le master
- "push", ce qui signifie que le master va "aller pousser" les dernières modifications vers les slaves

Qu'en déduisez-vous ?
