# Rôles / ACL en SQL

## Mise en place

Connectez-vous au serveur SQL dédié, soit via Workbench soit en ligne de commande : `mysql -u <user> -h <ip> -p <db>` en utilisant les paramètres données par votre enseignant•e.

Pourquoi, contrairement à l'option `-u` qui permet de fournir un utilisateur, l'option `-p` ne permet pas de fournir un mot de passe mais demande à l'utilisateur de saisir son mot de passe par la suite ?

Créez une table dont le nom sera votre nom de famille en minuscules sans accents, tirets ni espaces :

```sql
create table `<name>` (
  `id` int not null primary key auto_increment,
  `name` varchar(200) not null unique,
  `content` text not null
);
```

Tentez de voir la liste des tables : `show tables`. Que remarquez-vous ?

Créez un utilisateur dédié : `create user 'usr-<name>'@'%' identified by '<pwd>'`.
Tentez la connexion à la base `<db>` via cet utilisateur. Listez vos bases de données disponibles.

Que constatez-vous ?

## Permissions

> [!IMPORTANT]
> La syntaxe de `GRANT` est la suivante : `GRANT <perms> ON "<db>"."<table>" TO '<usr>'@'<host>'`.
> Dans cette liste :
> - `perms` est une des permissions SQL
> - `db` la base de données, on peut utiliser *
> - `table` la table concernée, on peut utiliser *
> - `usr` l'utilisateur concerné
> - `host` l'hôte par lequel l'utilisateur se connecte (localhost ou encore % pour "tous")
> 
> La syntaxe de `REVOKE` (pour retirer des droits) est la même en remplaçant `TO` par `FROM`.
>
> Deux droits en particulier sont spéciaux :
> - `USAGE` signifie "aucun droit"
> - `ALL` ou `ALL PRIVILEGES` signifie "tous les droits"

### Sélection / visualisation

Nous allons ajouter le droit d'accès (`SELECT`) à la base et à votre table en particulier.

Testez votre modification, puis listez vos tables. Que constatez-vous ?

> [!NOTE]
> Il est possible que vous ayiez besoin de "forcer" MySQL à rafraîchir son cache de permissions via `flush privileges`.

Tentez d'effectuer une insertion dans votre table, afin de vérifier l'application des droits d'accès.

### Droits cumulatifs

Ajoutez un droit intégral à l'user sur votre table, puis révoquez le droit de suppression.
Testez les différentes opérations d'insertion, mise à jour et suppression.

Testez de vidanger la table via truncate : que constatez-vous ?

Tentez de vindanger une autre table via truncate : quel est le message d'erreur ? Qu'en déduisez-vous ?

Quelle serait la commande pour restreindre / autoriser un truncate ?

Quels sont les dangers d'un privilège ALL sur une simple base de données ?

## Rôles

Les [rôles](https://mariadb.com/docs/server/security/user-account-management/roles/roles_overview) sont des regroupements de permissions, qui peuvent être appliquées pour des utilisateurs.

Ils sont factuellement gérés comme des utilisateurs virtuels regroupant zéro à plusieurs utilisateurs.

La syntaxe de CREATE ROLE est décrite dans la [documentation](https://mariadb.com/docs/server/reference/sql-statements/account-management-sql-statements/create-role).

# Activité

## Contexte

L'objectif est de mettre en place une API SQL de photographies et commentaires.

## Définition

Deux tables :
- pictures, contenant les références aux photographies
  - un titre
  - une URI
  - l'utilisateur l'ayant envoyé
- comments, contenant les commentaires
  - le contenu
  - la référence vers la photographie
  - l'utilisateur l'ayant envoyé

On gère les droits selon divers privilèges :
- les administrateurs ont tous les droits sur toutes les tables
- les artistes peuvent ajouter des photographies
- les modérateurs peuvent supprimer des commentaires
- les utilisateurs peuvent ajouter des commentaires
- tous les rôles peuvent sélectionner toutes les tables

## Création

Réalisez les requêtes de création des __rôles__ `admins`, `artists`, `mods` et `std`

## Tests de base

Testez vos différentes rôles en créant des utilisateurs SQL rattachés à chaque rôle. Pour remplir le champ `user`, il existe une fonction MySQL/MariaDB : `user()`.

Que remarquez-vous concernant ce champ ?

## Procédures stockées

Nous allons ajouter des procédures stockées pour gérer les différentes possibilités et simplifier certains éléments.

Quelques liens utiles :
- [Procédures stockées (création)](https://mariadb.com/docs/server/server-usage/stored-routines/stored-procedures/create-procedure)
- [Sécurité et procédures stockées](https://mariadb.com/docs/server/server-usage/stored-routines/stored-functions/stored-routine-privileges)

Créez la procédure stockée `addPicture`, qui permettra d'ajouter une photographie.
Cette procédure prendra en paramètre une URI et un titre et s'occupera de remplir l'utilisateur SQL ayant ajouté la photographie.

### Sécurité

En jouant sur le paramètre SECURITY lors de la création de la procédure stockée, comment pourriez-vous vous assurer que les artistes puissent uniquement créer des photographies avec leur nom d'utilisateur ? Quelles possibilités cela ouvre-t-il concernant les rôles et procédures ?

Réalisez le reste des procédures pour permettre l'ajout de commentaires, ainsi que la création automatisée d'utilisateurs.

### Vues et déclencheurs

Sur le même principe, analysez la manière dont les [vues (views)](https://mariadb.com/docs/server/server-usage/views) et [déclencheurs (triggers)](https://mariadb.com/docs/server/server-usage/triggers-events/triggers/trigger-overview) gèrent la sécurité. Quels sont les risques qui en découlent ?
