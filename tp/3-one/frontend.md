# II.1. Setup Frontend

![Frontend](./img/frontend.png)

Le Frontend est la machine qui contiendra la logique de l'application et qui va piloter les noeuds KVM.

**Toute cette partie est à réaliser uniquement sur `frontend.one`.**

## Sommaire

- [II.1. Setup Frontend](#ii1-setup-frontend)
  - [Sommaire](#sommaire)
  - [A. Database](#a-database)
  - [B. OpenNebula](#b-opennebula)
  - [C. Conf système](#c-conf-système)
  - [D. Test](#d-test)

## A. Database

🌞 **Installer un serveur MySQL**

- on va installer une version spécifique de MySQL, demandée par OpenNebula
- **ajouter un dépôt :**
  - télécharger le RPM disponible à l'URL suivante : https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm 
  - puis installez-le localement avec une commande `rpm`
- **installer le paquet qui contient le serveur MySQL depuis ce nouveau dépôt :**
  - faites un `dnf search mysql` pour voir les paquets dispos
  - vous devriez voir un paquet `mysql-community-server` dispo
  - installez-le


> Quand vous faites une commande `dnf install`, ça télécharge un `.rpm` (qui est juste une archive qui contient plein de fichiers) et ça l'installe automatiquement ("installer" ça veut juste dire qu'on extrait le contenu de l'archive et on dispose chaque fichier au "bon" endroit sur le système.)

🌞 **Démarrer le serveur MySQL**

- démarrer le service `mysqld`
- activer ce service au démarrage

🌞 **Setup MySQL**

- un mot de passe temporaire pour l'utilisateur `root` de la base de données a été généré, il est visible dans `/var/log/mysqld.log`
- connectez-vous à la base pour y effectuer les commandes SQL suivantes :

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED BY 'hey_boi_define_a_strong_password';
CREATE USER 'oneadmin' IDENTIFIED BY 'also_here_define_another_strong_password';
CREATE DATABASE opennebula;
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

## B. OpenNebula

🌞 **Ajouter les dépôts Open Nebula**

- ajoutez le dépôt suivant à la liste de dépôts de votre machine

```
[opennebula]
name=OpenNebula Community Edition
baseurl=https://downloads.opennebula.io/repo/6.8/RedHat/$releasever/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo2.key
gpgcheck=1
repo_gpgcheck=1
```

- puis effectuer un ptit `dnf makecache -y` pour accélérer la suite :)

🌞 **Installer OpenNebula**

- installez les paquets `opennebula`, `opennebula-sunstone`, `opennebula-fireedge`

🌞 **Configuration OpenNebula**

- dans le fichier `/etc/one/oned.conf`, définissez correctement les paramètres de connexion à la base de données, c'est la clause `DB =` que vous devez remplacer par

```conf
DB = [ BACKEND = "mysql",
       SERVER  = "localhost",
       PORT    = 0,
       USER    = "oneadmin",
       PASSWD  = "also_here_define_another_strong_password",
       DB_NAME = "opennebula",
       CONNECTIONS = 25,
       COMPARE_BINARY = "no" ]
```

🌞 **Créer un user pour se log sur la WebUI OpenNebula**

- pour ça, il faut se log en tant que l'utilisateur `oneadmin` sur le serveur
- une fois connecté en tant que `oneadmin`, inscrivez le user et le password de votre choix dans le fichier `/var/lib/one/.one/one_auth` sous la forme `user:password`

> Par exemple : vous stockez la chaîne `toto:super_password` dans le fichier.

🌞 **Démarrer les services OpenNebula**

- démarrez les services `opennebula`, `opennebula-sunstone`
- activez-les aussi au démarrage de la machine

## C. Conf système

🌞 **Ouverture firewall**

- ouvrez les ports suivants, avec des commandes `firewall-cmd` :

| Port  | Proto      | Why ?                        |
|-------|------------|------------------------------|
| 9869  | TCP        | WebUI (Sunstone)             |
| 22    | TCP        | SSH                          |
| 2633  | TCP        | Daemon `oned` et API XML RPC |
| 4124  | TPC et UDP | Monitoring                   |
| 29876 | TCP        | NoVNC proxy                  |

## D. Test

A ce stade, vous devriez déjà pouvoir **visiter la WebUI de OpenNebula.**

**RDV sur `http://10.3.1.11:9869`** avec un navigateur ! Vous pouvez vous log avec les identifiants définis dans la partie B.
