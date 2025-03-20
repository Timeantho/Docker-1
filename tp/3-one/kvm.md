# II.2. Noeuds KVM

![KVM bare-metal](./img/bare_kvm.png)

Dans un premier temps, **toute cette partie est à réaliser uniquement sur `kvm1.one`.**

Je vous recommande de faire ça sur un seul noeud d'abord, en entier, et une fois que ça fonctionne, vous ajouterez le deuxième (dans la partie IV. du TP).

## Sommaire

- [II.2. Noeuds KVM](#ii2-noeuds-kvm)
  - [Sommaire](#sommaire)
  - [A. KVM](#a-kvm)
  - [B. Système](#b-système)
  - [C. Ajout des noeuds au cluster](#c-ajout-des-noeuds-au-cluster)

## A. KVM

🌞 **Ajouter des dépôts supplémentaires**

- ajoutez les dépôts de OpenNebula, les mêmes que pour le Frontend !
  - **juste les dépôts**, n'installez pas les paquets OpenNebula
- ajoutez aussi les dépôts EPEL en exécutant :

```bash
dnf install -y epel-release
```

🌞 **Installer KVM**

- un paquet spécifique qui vient des dépôts OpenNebula : `opennebula-node-kvm`

🌞 **Démarrer le service `libvirtd`**

- activer le au démarrage de la machine aussi

## B. Système

🌞 **Ouverture firewall**

| Port | Proto | Why ? |
|------|-------|-------|
| 22   | TCP   | SSH   |
| 8472 | UDP   | VXLAN |

🌞 **Handle SSH**

- uniquement pour ce point, repassez en SSH sur `frontend.one`
- OpenNebula reposant sur des connexions SSH, elles doivent toutes se passer sans interaction humaine (pas de demande d'acceptation d'empreintes, ni de passwords par exemple)
  - donc, en étant connecté en tant que `oneadmin` sur `frontend.one`
  - vous **devez** pouvoir vous connecter vers `oneadmin@10.3.1.21` (`kvm1.one`) sans aucun prompt
- **une paire de clés SSH a été générée sur l'utilisateur `oneadmin`** (dans le dossier `.ssh` dans son homedir, comme d'hab)
- il faudra **déposer la clé publique sur les noeuds KVM**
  - la clé publique doit être déposée sur l'utilisateur `oneadmin` qui existe aussi sur les noeuds KVM
  - à la main ou `ssh-copy-id`
- il faudra aussi **trust les empreintes des autres serveurs**
  - à la main ou `ssh-keyscan`

## C. Ajout des noeuds au cluster

➜ **RDV de nouveau sur la WebUI de OpenNebula, et naviguez dans `Infrastructure > Hosts`**

Ajoutez le nouvel hôte KVM et assurez-vous qu'ils remontent en statut `ON`.
