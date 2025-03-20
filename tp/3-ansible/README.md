# TP3 : Automatisation et gestion de conf

**Dans ce TP, on bosse principalement sur Ansible.** On passe aussi sur du Azure en fin de TP pour les motiver, plutôt que de bosser en local.

**L'idée d'Ansible c'est :**

- on écrit de la conf une seule fois
- on peut déployer cette conf autant de fois qu'on veut sur autant de machines qu'on veut

> Par exemple, on décrit, avec une syntaxe Ansible, comment installer NGINX et le configurer, pour déployer par la suite autant de serveurs NGINX qu'on veut.

Généralement, on déploie depuis notre poste, donc on écrit tout le code Ansible sur notre propre poste. Et on tape une commande `ansible-playbook` pour déployer ça sur des machines.

![It's a fucking art](./img/its_a_fucking_art.jpg)

**Dans notre contexte :**

- Azure + TF automatise la création de VMs
  - le repackaging permet de pop des VMs avec des paquets pré-installés
- `cloud-init` pose la configuration élémentaire
  - en particulier : créer des users, conf sudo, déposer une clé publique
- **enfin, on passe Ansible**, pour déployer des services et de la conf supplémentaire
  - pour déployer par exemple un NGINX ou une base de données, etc

## Sommaire

- [TP3 : Automatisation et gestion de conf](#tp3--automatisation-et-gestion-de-conf)
  - [Sommaire](#sommaire)
- [I. Ansible first steps](#i-ansible-first-steps)
- [II. Range ta chambre](#ii-range-ta-chambre)
- [III. Repeat](#iii-repeat)
- [IV. Bonus : Aller plus loin](#iv-bonus--aller-plus-loin)
  - [1. Vault Ansible](#1-vault-ansible)
  - [2. Support de plusieurs OS](#2-support-de-plusieurs-os)

# I. Ansible first steps

*Ansible* sera donc notre outil de gestion de conf principal.

➜ Pour qu'Ansible fonctionne il nous faut :

- **du code Ansible**
  - tout au format YML, vous verrez je vous file tout pour le début :)
- **des machines sur lesquelles déposer de la conf**
  - ce sera des machines Linux dans Azure
  - Ansible et Python installés
    - installés avec `cloud-init`
    - en vrai y'a déjà Python by default sur la plupart des Linux)
  - un serveur SSH dispo (déjà en place avec les images proposées par Azure)
- **une machine qui possède le code Ansible**
  - ce sera votre PC
  - **donc il faut installer Ansible sur votre poste**, pas dans une VM (paraît que sous Windows c'est chiant, demandez-moi ptet avant, pour que je vous dise de passer sous Linux)
  - cette machine (votre PC donc) doit pouvoir se connecter sur les machines de destination en SSH
  - utilisation d'un utilisateur qui a accès aux droits `root` via la commande `sudo`

➜ **J'vous donne toutes les instructions petit à petit, [suivez le guide (document dédié à la partie I)](./ansible_first_steps.md) !**

# II. Range ta chambre

Bon ok c'est bien beau ce qu'on a fait jusque là, **mais c'est un peu crado de tout mettre dans un seul fichier `.yml`**, ou presque.

Quand on gère des dizaines, centaines, milliers de machines avec Ansible, c'est nécessaire de ranger un peu notre chambre, et d'**éclater ça dans plein de fichiers, de façon ordonnée et rigoureuse.**

> *La documentation officielle Ansible recommande plusieurs patterns. On va voir l'un des plus récurrents dans cette section.*

Idem, j'vous donne tout dans le doc dédié à la partie II, suivez-scrupuleusement les instructions ! :)

➜ **[Document dédié à la Partie II. Range ta chambre](./tidy_repo.md)**

# III. Repeat

Ptite section pour vous faire écrire un peu de code Ansible stylé.

➜ **[Document dédié à la Partie III. Repeat](./repeat.md)**

![Automate att](./img/automate_att.png)

# IV. Bonus : Aller plus loin

## 1. Vault Ansible

Afin de ne pas stocker de données sensibles en clair dans les fichiers Ansible, comme des mots de passe, on peut utiliser les [vault Ansible](https://docs.ansible.com/ansible/latest/user_guide/vault.html).

Cela permet de stocker ces données, mais dans des fichiers chiffrés, à l'intérieur du dépôt Ansible.

➜ **Utilisez les Vaults pour stocker les clés publiques des utilisateurs**

## 2. Support de plusieurs OS

**Il est possible qu'un rôle donné fonctionne pour plusieurs OS.** Pour ça, on va utiliser des conditions en fonction de l'OS de la machine de destination.

A chaque fois qu'on déploie de la conf sur une machine, cette dernière nous donne beaucoup d'informations à son sujet : ses ***facts***. Par exemple, on récupère la liste des cartes réseau de la machine, la liste des utilisateurs, l'OS utilisé, etc.

On peut alors récupérer ces variables dans nos tasks, pour les insérer dans des templates par exemple, ou encore effectuer du travail conditionnel :

```yml
  - name: Install apache
    apt: 
      name: apache
      state: latest
    when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'

  - name: Install apache
    yum: 
      name: httpd # le nom du paquet est différent sous CentOS
      state: latest
    when: ansible_distribution == 'CentOS'
```

➜ **Ajoutez une machine d'un OS différent à votre `Vagrantfile` et adaptez vos playbooks**

- passez sur une CentOS si vous étiez sur une base Debian jusqu'alors
- ou vice-versa
