# Part II : cloud-init

- [Part II : cloud-init](#part-ii--cloud-init)
  - [1. Intro](#1-intro)
  - [2. Gooooo](#2-gooooo)
  - [3. Write your own](#3-write-your-own)

## 1. Intro

![cloud-init](./img/cloudinit.jpg)

`cloud-init` est un outil qui permet de configurer une VM dès son premier boot.

C'est bien beau de pop une VM dans le "cloud", mais comment on dépose notre clé SSH ? On aimerait éviter de se co avec un password, et ce, dès la première connexion.

`cloud-init` a donc pour charge de configurer la VM **juste après son premier boot**. Il ne se lance **que** au premier boot.

Il peut par exemple :

- créer des users
  - définir des password
  - définir une conf `sudo`
  - poser une clé publique
- installer des paquets
- déposer des fichiers de conf
- démarrer des services
- [plein d'autres trucs](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)

Restez simples ici et utilisez une image stable et officielle comme `Ubuntu2204`, fournie par Azure : elle supporte `cloud-init` !

> Quand vous allez sur un site pour télécharger un OS et qu'il existe une version "cloud", généralement ça veut dire que c'est l'OS de base avec des trucs déjà faits dedans : y'a `cloud-init` qui est installé et prêt à run au prochain boot, un serveur SSH déjà installé et activé, une configuration `sudo`, etc. De quoi pouvoir instantanément se co à distance sur une machine préconfigurée quoi !

## 2. Gooooo

➜ **Sur votre PC, créez un fichier `cloud-init.txt` avec le contenu suivant :**

```yml
#cloud-config
users:
  - default
  - name: <TON_USER>
    sudo: false
    shell: /bin/bash
    ssh_authorized_keys:
      - <TA_CLE_PUBLIQUE>
```

🌞 **Tester `cloud-init`**

- en créant une nouvelle VM et en lui passant ce fichier `cloud-init.txt` au démarrage
- pour ça, utilisez une commande `az vm create`
- utilisez l'option `--custom-data /path/to/cloud-init.txt`

🌞 **Vérifier que `cloud-init` a bien fonctionné**

- connectez-vous en SSH à la VM nouvellement créée, directement sur le nouvel utilisateur créé par `cloud-init`

## 3. Write your own

🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**

- installer Docker sur la machine
- ajoutez un user qui porte votre pseudo
  - il a un password défini
  - clé SSH publique déposée
  - il a accès aux droits de `root` via `sudo`
  - membre du groupe `docker`
- l'image Docker `alpine:latest` doit être téléchargée

---

➜ Shortcut to [**Part III** : Terraform](part3.md)
