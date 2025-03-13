# Part III : `docker-compose`

## Sommaire

- [Part III : `docker-compose`](#part-iii--docker-compose)
  - [Sommaire](#sommaire)
  - [1. Intro](#1-intro)
  - [2. WikiJS](#2-wikijs)
  - [3. Make your own meow](#3-make-your-own-meow)

## 1. Intro

**`docker compose` est un outil qui permet de lancer plusieurs conteneurs en une seule commande.**

![not enough](./img/compose.jpg)

> En plus d'être pratique, il fournit des fonctionnalités additionnelles, liés au fait qu'il s'occupe à lui tout seul de lancer tous les conteneurs. On peut par exemple demander à un conteneur de ne s'allumer que lorsqu'un autre conteneur est devenu "healthy". Idéal pour lancer une application après sa base de données par exemple.

Le principe de fonctionnement de `docker compose` :

- on écrit un fichier qui décrit les conteneurs voulus
  - c'est le `docker-compose.yml`
  - tout ce que vous écriviez sur la ligne `docker run` peut être écrit sous la forme d'un `docker-compose.yml`
- on se déplace dans le dossier qui contient le `docker-compose.yml`
- on peut utiliser les commandes `docker compose` :

```bash
# Allumer les conteneurs définis dans le docker-compose.yml
$ docker compose up
$ docker compose up -d

# Eteindre
$ docker compose down

# Explorer un peu le help, il y a d'autres commandes utiles
$ docker compose --help
```

La syntaxe du fichier peut par exemple ressembler à :

```yml
version: "3.8"

services:
  db:
    image: mysql:5.7
    restart: always
    ports:
      - '3306:3306'
    volumes:
      - "./db/mysql_files:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: beep

  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    restart: unless-stopped
```

> Petite astuce : les noms déclarés dans le `docker-compose.yml` sont joignables sur le réseau par les conteneurs. Dans le cas du fichier au dessus, si on pop un  shell dans le conteneur `nginx` et qu'on fait `ping db`, ça fonctionne :d

## 2. WikiJS

WikiJS est une application web plutôt cool qui comme son nom l'indique permet d'héberger un ou plusieurs wikis. Même principe qu'un MediaWiki donc (solution opensource utilisée par Wikipedia par exemple) mais avec un look plus moderne.

🌞 **Installez un WikiJS** en utilisant Docker

- WikiJS a besoin d'une base de données pour fonctionner
- il faudra donc deux conteneurs : un pour WikiJS et un pour la base de données
- référez-vous à la doc officielle de WikiJS, c'est tout guidé

🌞 **Call me** when it's done

- je dois pouvoir visiter votre WikiJS (il doit être dispo sur internet)

## 3. Make your own meow

Ici on se rapproche d'un cas d'utilisation réel : je vous mets une application sur les bras et vous devez la conteneuriser. 

L'application : 

- codée en `python3` avec mes ptites mimines
  - [les sources sont dispos ici](./python-app)
    - info : elle écoute sur le port 8888/tcp
    - c'est une app web, vous pourrez visitez avec votre navigateur
  - nécessite des librairies installables avec `pip`
    - `pip install -r requirements.txt`
- a besoin d'un Redis pour fonctionner
  - c'est une base de donnée (pour faire simple :d)
  - vous devez donc ajouter un autre conteneur dans le `docker-compose.yml`
  - il doit être joignable sur le nom `db` (port par défaut 6379/TCP)
  - utilisez [l'image Redis officielle](https://hub.docker.com/_/redis)

🌞 **Vous devez :**

- construire une image qui
  - contient `python3`
  - contient l'application et ses dépendances
  - lance l'application au démarrage du conteneur
- écrire un `docker-compose.yml` qui définit le lancement de deux conteneurs :
  - l'app python
  - le Redis dont il a besoin
