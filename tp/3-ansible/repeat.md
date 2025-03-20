# III. Repeat

Ici, vous écrivez un peu plus de code Ansible par vous-mêmes. GLHF.

## Sommaire

- [III. Repeat](#iii-repeat)
  - [Sommaire](#sommaire)
  - [1. NGINX](#1-nginx)
  - [2. Common](#2-common)
  - [3. Dynamic loadbalancer](#3-dynamic-loadbalancer)

## 1. NGINX

➜ **On reste dans le rôle `nginx`**, faites en sorte que :

- on puisse déclarer la liste `vhosts` en *host_vars*
- si cette liste contient plusieurs `vhosts`, le rôle les déploie tous (exemple en dessous)
- le port précisé est automatiquement ouvert dans le firewall
- vous gérez explicitement les permissions de tous les fichiers

Exemple de fichier de variable avec plusieurs Virtual Hosts dans la liste `vhosts` :

```yml
vhosts:
  - test2:
    nginx_servername: test2
    nginx_port: 8082
    nginx_webroot: /var/www/html/test2
    nginx_index_content: "<h1>teeeeeest 2</h1>"
  - test3:
    nginx_servername: test3
    nginx_port: 8083
    nginx_webroot: /var/www/html/test3
    nginx_index_content: "<h1>teeeeeest 3</h1>"
```

➜ **Ajoutez une mécanique de `handlers/`**

- c'est un nouveau dossier à placer dans le rôle
- je vous laisse découvrir la mécanique par vous-mêmes et la mettre en place
- vous devez trigger un *handler* à chaque fois que la conf NGINX est modifiée
- vérifiez le bon fonctionnement
  - vous pouvez voir avec un `systemctl status` depuis quand une unité a été redémarrée

🌞 **En compte-rendu...**

- tous les fichiers modifiés/ajoutés

## 2. Common

➜ **On revient sur le rôle `common`**, les utilisateurs déployés **doivent** :

- avoir un password
- avoir un homedir
- avoir accès aux droits de `root` *via* `sudo`
- être dans un groupe `admin`
- avoir une clé SSH publique déposé dans leur `authorized_keys`

> Toutes ces données doivent être stockées dans les `group_vars`.


🌞 **En compte-rendu...**

- tous les fichiers modifiés/ajoutés

## 3. Dynamic loadbalancer

➜  **Créez un nouveau rôle : `webapp`**

- ce rôle déploie une application Web de votre choix, peu importe
- elle déploie aussi le serveur web nécessaire pour que ça tourne
  - vous pouvez clairement réutiliser le rôle NGINX d'avant qui déploie une bête page HTML

> Vraiment, peu importe, une bête page HTML, ou un truc open source comme un NextCloud. Ce qu'on veut, c'est simplement une interface visible.

➜  **Créez un nouveau rôle : `rproxy` (pour *reverse proxy*)**

- ce rôle déploie un NGINX
- NGINX est automatiquement configuré pour agir comme un reverse proxy vers une liste d'IP qu'on lui fournit en variables
  - à priori, vous allez gérer ça avec des `host_vars` et `group_vars`

➜ **Effectuez le déploiement suivant :**

- deux machines portent le rôle `webapp`
- une machine porte le rôle `rproxy`

> **Il sera nécessaire d'ajouter une nouvelle VM à votre déploiement Vagrant !**

- faites en sorte que :
  - si on déploie une nouvelle machine qui porte le rôle `webapp`, la conf du reverse proxy se met à jour en fonction
  - si on supprime une machine `webapp`, la conf du reverse proxy se met aussi à jour en fonction

> La configuration de votre loadbalancer devient dynamique, et plus aucune connexion manuelle n'est nécessaire pour ajuster la taille du parc en fonction de la charge.

➜ **Exemple de configuration NGINX** qui effectue un loadbalancing entre deux machines serveurs Web qui portent les IP `192.168.56.102` et `192.168.56.103` (ce fichier est à placer dans le dossier `/etc/nginx/conf.d/` et doit porter l'extension `.conf` afin d'être inclus par le fichier `/etc/nginx/nginx.conf`) :

```NGINX
upstream super_application {
    server 192.168.56.102;
    server 192.168.56.103;
}

server {
    server_name anyway.com;

    location / {
        proxy_pass http://super_application;
        proxy_set_header    Host $host;
    }
}
```

🌞 **En compte-rendu...**

- tous les fichiers modifiés/ajoutés
