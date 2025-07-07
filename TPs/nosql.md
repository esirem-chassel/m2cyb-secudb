# Sécurité des bases NoSQL

Lancez une [installation basique de MongoDB](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-debian/).
Attention à bien suivre les étapes indiquées.

## Accès par défaut

A la fin de l'installation, soyez assurés de tenter la commande `mongosh` sans droits administrateurs (donc sans sudo). Que se passe-t-il ?
Qu'en déduisez-vous ?

## Binding

Par défaut, MongoDB est "bind" sur `127.0.0.1`, ce qui signifie qu'il n'accepte que des connexions venant du poste local.

Permettez la connexion depuis le réseau local (le réseau universitaire), puis tentez d'y accéder depuis un autre poste, via mongosh et l'argument `--host`.

Que remarquez-vous ?

> [!Caution]
> Toute modification de la configuration de MongoDB via modification des fichiers de configuration nécessitent à minima un reload, ou encore un restart du service.

## Port

De même que pour le binding, le port par défaut de MongoDB peut être changé, pour, tout en permettant l'accès depuis l'extérieur de la machine, accroître la sécurité.

Ces ports sont indiqués sur la page de listing des ports par défaut. Celui qui nous intéresse particulièrement est le port utilisé pour `mongod`, le daemon de MongoDB.

Modifiez ce port par défaut, et redémarrez le service.

### nmap

Depuis un autre poste, effectuez un nmap vers la machine où votre MongoDB est installé.
Une fois la commande réalisée, vous obtiendrez une liste de ports derrière lesquels un service est accessible.
Tentez de lancer [un script utilitaire de nmap pour obtenir diverses informations](https://nmap.org/nsedoc/scripts/mongodb-info.html) sur un ou plusieurs de ces ports.

Qu'en déduisez-vous ?

## Restrictions de sécurité

Contrairement à la majorité des bases SQL, les bases NoSQL sont souvent plus "ouvertes" dans leur configuration par défaut,
une conséquence logique de leur cadre régulier d'utilisation (APIs internes, bases pour outil en local...).
Cela permet à l'outil de d'abord se reposer sur les mécaniques systèmes avant d'implémenter des limites et des contrôles qui pourraient interférer avec ces cas d'usage.

MongoDB fournit différemment mécanismes de sécurité, pour restreindre ses accès en plus des restrictions "classiques" (binding de ports, écoute de réseau...).

Une [liste de vérification](https://www.mongodb.com/docs/manual/administration/security-checklist/) est présente dans la documentation officielle.

### Mode Authentifié

Il existe différents modes d'authentification, et il est nécessaire d'en choisir au moins un pour activer le mode authentifié sur un serveur MongoDB.

Le mode par défaut, [SCRAM](https://www.mongodb.com/docs/manual/core/security-scram/), est en réalité un mode proche des authentifications classiques SQL;
il s'agit d'utilisateurs dédies, stockés au niveau du système de bases de données, et pouvant être manipulés par ce même système.

> [!Note]
> La majorité des autres modes d'authentification sont limités à la version Entreprise, donc payante, de MongoDB.

En suivant [la page dédiée](https://www.mongodb.com/docs/manual/tutorial/configure-scram-client-authentication/), activez l'authentification avec mode SCRAM et créez deux utilisateurs dédiés :
- un admin capable de se connecter à tout
- un utilisateur "standard" capable de se connecter à une base précise

> [!Tip]
> N'hésitez pas à vous référer à la [documentation de la méthode `createUser`](https://www.mongodb.com/docs/manual/reference/method/db.createUser/) au besoin !

> [!Caution]
> Vous aurez besoin de cet utilisateur administrateur.
> La façon dont MongoDB Auth+SCRAM fonctionne est de verrouiller les accès dès l'ajout du paramètre `authorization`.
> Cela signifierait que si vous n'avez aucun utilisateur créé au démarrage, vous ne pourrez plus vous connecter.
> Il est à noter qu'[il existe un cas très particulier de connexion via localhost quand aucun utilisateur n'existe encore](https://www.mongodb.com/docs/manual/core/localhost-exception/#std-label-localhost-exception).
> Cela reste une exception.


### Restriction par rôle

Vous l'avez peut-être vu lorsque vous avez manipulé les utilisateurs dans la section précédente, mais une notion de rôles existe également en MongoDB.

Il existe à la fois des [rôles par défaut](https://www.mongodb.com/docs/manual/reference/built-in-roles/),
ainsi que des [rôles que vous pouvez créer](https://www.mongodb.com/docs/manual/core/security-user-defined-roles/),
à l'instar des SGBDR classiques.

Chaque rôle est une collection de [privilèges / actions](https://www.mongodb.com/docs/manual/reference/privilege-actions/) que l'on peut réaliser,
et par conséquences, quelles fonctions peuvent être utilisées sur la ou les bases impactées.

Via la manpulation de ces rôles, effectuez des tests quant à votre utilisateur standard créé précédemment pour autoriser / interdire des actions
comme la recherche, l'insertion ou la suppression d'éléments...

## Injection (No)SQL

> [!Note]
> Pour des besoins pratiques, nous allons "simuler" cette injection dans la console `mongosh`.
> Si vous souhaitez réaliser un script susceptible d'être exploité de la même manière,
> vous devrez choisir un langage, installer le driver correspondant, et développer le script en question.
> Des langages appropriés peuvent être par exemple Python (Django...) ou encore JS (Express...).

Soit une table users, utilisée pour la connexion d'utilisateurs à une application dédiée.
Cette table comporte notemment deux propriétés : user et pwd (explicites).
La vérification est réalisée via une ligne semblable à `db.users.findOne({ user: inputUser, pwd: inputPwd })`.

Que pourriez-vous passer comme valeur à `inputUser` et `inputPwd` pour vous assurer d'une connexion ?

<!-- user: { "$ne": null } , pwd: { "$ne": null } -->

## Analyse des logs

Où sont situés les logs de l'applicatif ?
Que contiennent-ils ?

# Annexes

## Base fictive

```js
db.books.insertMany([
  {
    title: "The Empire Reborn",
    author: "Lara N. Vex",
    published: 2245,
    series: "Galactic Rebellion",
    volume: 1,
    genres: ["space opera", "adventure", "war"],
    available: true,
    rating: 4.7,
    pages: 382,
    tags: ["empire", "rebellion", "battle"],
    language: "English"
  },
  {
    title: "Echoes of Andromeda",
    author: "Thalor Venn",
    published: 2250,
    series: "Andromeda Echo",
    volume: 2,
    genres: ["space opera", "mystery", "exploration"],
    available: false,
    rating: 4.3,
    pages: 410,
    tags: ["exploration", "first contact"],
    language: "English"
  },
  {
    title: "Starlight Exodus",
    author: "J.K. Klyne",
    published: 2260,
    series: "Void Exodus",
    volume: 1,
    genres: ["space opera", "survival", "sci-fi"],
    available: true,
    rating: 4.9,
    pages: 512,
    tags: ["survival", "AI", "colonization"],
    language: "English"
  },
  {
    title: "The Last Orbit",
    author: "Vera Kest",
    published: 2238,
    genres: ["space opera", "drama"],
    available: true,
    rating: 4.0,
    pages: 299,
    tags: ["lost crew", "black hole"],
    language: "English"
  },
  {
    title: "Chronicles of Vega Prime",
    author: "Darin Sol",
    published: 2255,
    series: "Vega Prime",
    volume: 3,
    genres: ["space opera", "political intrigue"],
    available: false,
    rating: 4.5,
    pages: 460,
    tags: ["politics", "betrayal", "star system"],
    language: "English"
  }
]);
```
