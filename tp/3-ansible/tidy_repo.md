# II. Range ta chambre

Voyons donc dans cette partie la structure d'un dépôt Ansible clean.

## Sommaire

- [II. Range ta chambre](#ii-range-ta-chambre)
  - [Sommaire](#sommaire)
  - [1. Structure du dépôt : inventaires](#1-structure-du-dépôt--inventaires)
  - [2. Structure du dépôt : rôles](#2-structure-du-dépôt--rôles)
  - [3. Structure du dépôt : variables d'inventaire](#3-structure-du-dépôt--variables-dinventaire)
  - [4. Structure du dépôt : rôle avancé](#4-structure-du-dépôt--rôle-avancé)

## 1. Structure du dépôt : inventaires

➜ **Dans votre répertoire de travail Ansible...**

- créez un répertoire `inventories/`
- créez un répertoire `inventories/vagrant_lab/`
- déplacez le fichier `hosts.ini` dans `inventories/vagrant_lab/hosts.ini`
- assurez vous que pouvez toujours déployer correctement avec une commande `ansible-playbook`

## 2. Structure du dépôt : rôles

**Les *rôles* permettent de regrouper de façon logique les différentes configurations qu'un dépôt Ansible contient.**

Un *rôle* correspond à une configuration spécifique, ou une application spéficique. Ainsi on peut trouver un rôle `apache` qui installe le serveur web Apache, ou encore `mysql`, `nginx`, etc.

**Un rôle doit être générique.** Il ne doit pas être spécifique à telle ou telle machine.

Il existe des conventions et bonnes pratiques pour structurer les *rôles* Ansible, que nous allons voir dans cette partie.

On crée souvent un *rôle* `common` qui est appliqué sur toutes les machines du parc, et qui pose la configuration élémentaire, commune à toutes les machines.

➜ **Ajout d'un fichier de config Ansible**

- dans le répertoire de travail, créez un fichier `ansible.cfg` :

```ini
[defaults]
roles_path = ./roles
```

➜ **Dans votre répertoire de travail Ansible...**

- créez un répertoire `roles/`
- créez un répertoire `roles/common/`
- créez un répertoire `roles/common/tasks/`
- créez un fichier `roles/common/tasks/main.yml` avec le contenu suivant :

```yml
- name: Install common packages
  import_tasks: packages.yml
```

- créez un fichier `roles/common/tasks/packages.yml` :
  - on va en profiter pour manipuler des variables Ansible

```yml
- name: Install common packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  with_items: "{{ common_packages }}" # ceci permet de boucler sur la liste common_packages
```

- créez un répertoire `roles/common/defaults/`
- créez un fichier `roles/common/defaults/main.yml` :

```yml
common_packages:
  - vim
  - git
```

- créez un fichier `playbooks/main.yml`

```yml
- hosts: tp4
  roles:
    - common
```

➜ **Testez d'appliquer ce playbook avec une commande `ansible-playbook`**

## 3. Structure du dépôt : variables d'inventaire

Afin de garder la complexité d'un dépôt Ansible sous contrôle, il est récurrent d'user et abuser de l'utilisation des variables.

Il est possible dans un dépôt Ansible de déclarer à plusieurs endroits : on a déjà vu le répertoire `defaults/` à l'intérieur d'un rôle (comme notre `roles/common/defaults/`) que l'on a créé juste avant. Ce répertoire est utile pour déclarer des variables spécifiques au rôle.

Qu'en est-il, dans notre cas présent, si l'on souhaite installer un paquet sur une seule machine, mais qui est considéré comme un paquet de "base" ? On aimerait l'ajouter dans la liste dans `roles/common/defaults/main.yml` mais ce serait moche d'avoir une condition sur le nom de la machine à cet endroit (un rôle doit être générique).

**Pour cela on utilise les `host_vars`.**

➜ **Dans votre répertoire de travail Ansible...**

- créez un répertoire `inventories/vagrant_lab/host_vars/`
- créez un fichier `inventories/vagrant_lab/host_vars/10.3.1.11.yml` :

> Le nom de ce fichier YML doit être rigoureusement et strictement identique au nom tel qu'il a été déclaré dans le fichier d'inventaire (`hosts.ini`). Donc, nous qui avons déclaré l'hôte grâce à son IP 10.3.1.11 il faut donc nommer le fichier `10.3.1.11.yml`.

```yml
common_packages:
  - vim
  - git
  - rsync
```

➜ **Testez d'appliquer le playbook avec une commande `ansible-playbook`**

---

Il est aussi possible d'attribuer des variables à un groupe de machines définies dans l'inventaire. **On utilise pour ça les `group_vars`.**

➜ **Dans votre répertoire de travail Ansible...**

- créez un répertoire `inventories/vagrant_lab/group_vars/`
- créez un fichier `inventories/vagrant_lab/group_vars/tp4.yml` :

> Le nom de ce fichier YML doit être rigoureusement et strictement identique au nom tel qu'il a été déclaré dans le fichier d'inventaire (`hosts.ini`). Donc, nous qui avons déclaré le groupe d'hôtes grâce au nom `[tp4]`il faut donc nommer le fichier `tp4.yml`.

```yml
users:
  - le_nain
  - l_elfe
  - le_ranger
```

➜ **Modifiez le fichier `roles/common/tasks/main.yml`** pour inclure un nouveau fichier  `roles/common/tasks/users.yml` :

- il doit utiliser cette variable `users` pour créer des utilisateurs
- réutilisez la syntaxe avec le `with_items`
- la variable `users` est accessible, du moment que vous déployez sur les machines qui sont dans le groupe `tp4`

➜ **Vérifiez la bonne exécution du playbook**

## 4. Structure du dépôt : rôle avancé

➜ **Créez un nouveau rôle `nginx`**

- créez le répertoire du rôle `roles/nginx/`
- créez un sous-répertoire `roles/nginx/tasks/` et un fichier `main.yml` à l'intérieur :

```yml
- name: Install NGINX
  import_tasks: install.yml

- name: Configure NGINX
  import_tasks: config.yml

- name: Deploy VirtualHosts
  import_tasks: vhosts.yml
```

➜ **Remplissez le fichier `roles/nginx/tasks/install.yml`**

- il doit installer le paquet NGINX
  - je vous laisse gérer :)

➜ **On va y ajouter quelques mécaniques : fichiers et templates :**

- créez un répertoire `roles/nginx/files/`
- créez un fichier `roles/nginx/files/nginx.conf`
  - récupérez un fichier `nginx.conf` par défaut (en faisant une install à la main par exemple)
  - ajoutez une ligne `include conf.d/*.conf;`
- créez un répertoire `roles/nginx/templates/`
- créez un fichier `roles/nginx/templates/vhost.conf.j2` :
  - `.j2` c'pour Jinja2, c'est le nom du moteur de templating utilisé par Ansible

```nginx
server {
        listen {{ nginx_port }} ;
        server_name {{ nginx_servername }};

        location / {
            root {{ nginx_webroot }};
            index index.html;
        }
}
```

➜ **Remplissez le fichier `roles/nginx/tasks/config.yml`** :

```yml
- name : Main NGINX config file
  copy:
    src: nginx.conf # pas besoin de préciser de path, il sait qu'il doit chercher dans le dossier files/
    dest: /etc/nginx/nginx.conf
```

➜ **Quelques variables `roles/nginx/defaults/main.yml`** :

```yml
nginx_servername: test
nginx_port: 8080
nginx_webroot: /var/www/html/test
nginx_index_content: "<h1>teeeeeest</h1>"
```

➜ **Remplissez le fichier `roles/nginx/tasks/vhosts.yml`** :

```yml
- name: Create webroot
  file:
    path: "{{ nginx_webroot }}"
    state: directory

- name: Create index
  copy:
    dest: "{{ nginx_webroot }}/index.html"
    content: "{{ nginx_index_content }}"

- name: NGINX Virtual Host
  template:
    src: vhost.conf.j2
    dest: /etc/nginx/conf.d/{{ nginx_servername }}.conf
```

➜ **Deploy !**

- ajoutez ce rôle `nginx` au playbook
- et déployez avec une commande `ansible-playbook`

![That feel](./img/that_feel.jpg)
