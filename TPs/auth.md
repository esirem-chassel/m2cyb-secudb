# Auth

## Notes préalables

Pour cette activité, vous aurez besoin de MariaDB/MySQL et PostGreSQL.

## Rappel de définition

Dans la plupart des SGBDR client/serveur, le schéma est le suivant :
- le client se connecte à un serveur (hôte) via un port d'entrée (souvent négligé) en spécifiant des identifiants de connexion (selon la méthode de connexion)
- le serveur valide (ou non) la connexion
- si une base de données a été spécifiée à la connexion, l'accès à celle-ci est vérifié (et possiblement validé)

Il est à noter que SQLite ne fournit pas le même schéma d'accès, s'agissant d'une base "tout intégré", en un seul fichier, donc sans serveur.

Pour les trois SGBDR étudiés dans ce document (MariaDB, MySQL, PostGreSQL), il existe plusieurs méthodes d'authentification.

## Password

L'authentification par mot de passe est un classique en bases de données traditionnelles.
Lors de la création d'un utilisateur, celui-ci est autorisé pour un utilisateur et un nom d'hôte, avec éventuellement (généralement) un mot de passe.

Lors de la connexion, une vérification est réalisée sur l'hôte, selon ce qui a été demandé. L'hôte est traditionnellement fourni en ligne de commande via une option `-h` et fait partie intégrante des chaînes de connexion dites DSN.

Cela permet notemment d'interdire / autoriser des connexions à travers des réseaux. Bloquer l'utilisateur sur "localhost", par exemple, assure que cet utilisateur ne puisse se connecter au serveur sans être d'abord connecté à la machine. Ce blocage peut être réalisé sur la base d'une chaîne littérale, utilisant des jokers (comme LIKE) ou via la définition d'un réseau, en utilisant l'adresse de réseau ainsi que le masque de sous-réseaux correspondant.

Testez la création de deux utilisateurs :
```sql
-- PostgreSQL
CREATE USER testuser WITH PASSWORD 'simplepass';

-- MySQL
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'simplepass';
```

> [!Note]
> La commande pour supprimer un utilisateur est `DROP USER`. N'hésitez pas à créer et supprimer des utilisateurs dans le cadre de cette activité.

Testez la connexion de vos utilisateurs, d'abord sans host, puis en spécifiant un host :
- localhost
- 127.0.0.1
- `<votre ip locale>`

### Réseau

En utilisant la [documentation MariaDB sur le sujet](https://mariadb.com/docs/server/reference/sql-statements/account-management-sql-statements/create-user#account-names), tentez d'autoriser la connexion à votre base MySQL/MariaDB pour votre réseau local.

## No-pwd

Il existe dans de nombreux SGBDR un moyen de connecter l'utilisateur en le confrontant à un compte système correspondant.
En terme de sécurité, cela signifie qu'il est nécessaire d'avoir d'abord accès au compte Linux/Unix/Windows correspondant, afin de pouvoir se connecter à une base de données.

Certains SGBDR, comme SQLite, ne gèrent pas l'aspect sécuritaire et laisse donc au système (ou à n'importe quel composant du système) de gérer cet aspect, ce qui peut revenir au même effet : en gérant des droits d'accès Linux à un fichier SQLite, vous cloisonnez son accès à un utilisateur ou groupe spécifique.

### Unix sockets

Une méthode de connexion utilisable dans MariaDB et PostGreSQL s'appuie sur les sockets Unix, disponibles dans tout système Linux / Unix, permettant de "lier" un utilisateur à un processus, et donc de vérifier l'utilisateur actuellement connecté.

#### MariaDB

Sous MariaDB, un [plugin](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-unix-socket) nommé `unix_socket` permet de réaliser cela.

Son usage est simple :

```sql
CREATE USER username@hostname IDENTIFIED VIA unix_socket;
```

Ainsi, seul un utilisateur Linux / Unix nommé "username" pourra se connecter via l'hôte "hostname".

Il existe de nombreux avantages mais aussi des conséquences à ce comportement. Consultez la documentation pour de plus amples informations. L'immense majorité des implications proviennent de la nécessité, logique, à fournir un utilisateur Linux / Unix pour permettre cette connexion.

#### PostGreSQL

Sous PostGreSQL, un [grand nombre de méthode d'authentification existent](https://www.postgresql.org/docs/current/auth-methods.html). Chaque méthode a ses avantages et inconvénients, ainsi que ses cas d'usage. Même la méthode `trust` peut avoir des cas d'usage... A condition, évidemment, de conjuguer cela à des rôles précis.

La configuration des éléments d'authentification sous PostGreSQL passe par un fichier traditionnellement nommé "ph_hba.conf". La syntaxe est décrite dans la [documentation associé](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html).

Sous MariaDB et PostGreSQL, ajoutez un utilisateur lié à votre utilisateur Linux actuel et testez l'accès.

### PAM

PAM (Pluggable Authentication Modules) est un mécanisme intégré (semblable à une API) pour fournir un moyen d'interconnexion d'authentification avec le système, passant par des modules PAM. Concrètement, pour les différents SGBDR, cela revient à laisser au système le rôle de l'authentification.

Dans le principe, un même module PAM devrait être capable de gérer la connexion via PAM, que ce soit pour un système Linux ou Windows. Dans la pratique, le module est nommé `.so` sous Linux et `.dll` sous Windows, par exemple.

PostGreSQL, MariaDB et MySQL supportent tous trois l'authentification par PAM, parfois via un module délivré à part. Comme PAM est d'abord une abstraction, cela signifie que la source d'authentification peut être le système lui-même ou un autre mécanisme (LDAP par exemple).

Si l'utilisation et le lien entre PAM et les Unix Sockets peuvent sembler logiques, ce sont bien deux mécanismes différents.

Sous MariaDB, [installez et configurez le module PAM](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-with-pluggable-authentication-modules-pam/configuring-pam-authentication-and-user-mapping-with-unix-authentication) afin d'ajouter l'authentification pour un utilisateur.

### Trust

Sous PostGreSQL, ajoutez un utilisateur dédié via ccès "trust" et testez l'accès également.

### Plugins supplémentaires

MySQL a introduit différents plugins au fil de son existence (la grande spécialité de MySQL était historiquement sa flexibilité : moteurs, plugins...).
Par exemple, un des plugins [gérait directement la gestion de l'authentification basés sur SHA1](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-mysql_native_password). Une [version antérieure de cette méthode existe toujours](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-mysql_old_password).

En suivant la documentation associée, créez un nouvel utilisateur en se basant sur l'ancienne méthode OLD_PASSWORD. Vérifiez, dans les tables systèmes de MySQL/MariaDB, le mot de passe en question. Que constatez-vous ?

## Host-based

A chaque définition d'un utilisateur, un second paramètre a été ajouté : l'hôte. Traditionnellement après l'utilisateur, séparé par un `@`, l'information d'hôte correspond à un fitrage supplémentaire selon l'hôte contacté par le client.

Ce filtrage permet de valider ou non la connexion, en plus de l'utilisateur. En effet, si coté réseau, le client n'a pas accès à l'hôte, alors celui-ci sera déboûté avant même d'arriver sur le serveur du SGBDR. De plus, un même serveur SGBDR peut gérer plusieurs hôtes différents, et réagir différemment selon si la connexion vient de la même machine (donc à destination de localhost ou 127.0.0.1) ou d'une machine du même réseau (en utilisant l'adresse IP locale au réseau du serveur) ou encore d'une adresse externe (en utilisant une IP publique ou un nom de domaine).

### From grants (MariaDB/MySQL)

Coté utilisateurs MySQL/MariaDB, l'hôte autorisé fait partie intégrante de l'utilisateur. Lorsque vous ajoutez un utilisateur ou effectuez un GRANT, l'utilisateur concerné spécifie l'hôte associé. VOir la documentation associé [sous MariaDB](https://mariadb.com/docs/server/reference/sql-statements/account-management-sql-statements/create-user#account-names) et [sous MySQL](https://dev.mysql.com/doc/refman/8.4/en/account-names.html).

Sous MySQL/MariaDB, ouvrez la possibilité à un utilisateur de se connecter à travers le réseau, et vérifiez le résultat avec votre enseignant.

### From config (PostGreSQL)

Coté utilisateurs PostGreSQL, la vérification ne s'applique plus au niveau du GRANT mais de la configuration. Chaque ligne de configuration d'authentification PostGreSQL peut embarquer une partie host/hostaddr pour spécifier un ou plusieurs hôtes acceptés pour cet utilisateur.

Sous PostGreSQL, ouvrez la possibilité à un utilisateur de se connecter à travers le réseau, et vérifiez le résultat avec votre enseignant.
