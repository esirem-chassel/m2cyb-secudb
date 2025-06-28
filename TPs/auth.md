# Auth

## Notes préalables

Pour cette activité, vous aurez besoin de MariaDB/MySQL et PostGreSQL.

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

### Unix sockets

### PAM ?

## Host-based

### From grants
