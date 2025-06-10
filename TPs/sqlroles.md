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

## Rôles

> [!IMPORTANT]
> La syntaxe de `GRANT` est la suivante : `GRANT <perms> ON "<db>"."<table>" TO '<usr>'@'<host>`.
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

Nous allons ajouter le droit d'accès (`SELECT`) à la base et à votre table en particulier.

Testez votre modification, puis listez vos tables. Que constatez-vous ?

> [!NOTE]
> Il est possible que vous ayiez besoin de "forcer" MySQL à rafraîchir son cache de permissions via `flush privileges`.



