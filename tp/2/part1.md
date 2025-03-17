# Part I : Programmatic approach

On commence tranquillement avec cette première partie !

- [Part I : Programmatic approach](#part-i--programmatic-approach)
- [I. Premiers pas](#i-premiers-pas)
- [II. Un ptit LAN](#ii-un-ptit-lan)

# I. Premiers pas

🌞 **Créez une VM depuis le Azure CLI**

- en utilisant uniquement la commande `az` donc
- assurez-vous que dès sa création terminée, vous pouvez vous connecter en SSH en utilisant une IP publique
- vous devrez préciser :
  - quel utilisateur doit être créé à la création de la VM
  - le fichier de clé utilisé pour se connecter à cet utilisateur
  - comme ça, dès que la VM pop, on peut se co en SSH !
- je vous laisse faire vos recherches pour créer une VM avec la commande `az`

> *Encore une fois, je vous recommande d'utiliser `az interactive`.*

Par exemple, une commande simple pour faire ça (ça suppose qu'une clé publique SSH existe dans `../.ssh/id_rsa.pub`):

```bash
az vm create -g meo -n super_vm --image Ubuntu2204 --admin-username it4 --ssh-key-values ../.ssh/id_rsa.pub
```

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.**

- une fois connecté, observez :
  - **la présence du service `walinuxagent`**
    - permet à Azure de monitorer et interagir avec la VM
  - **la présence du service `cloud-init`**
    - permet d'effectuer de la configuration automatiquement au premier lancement de la VM
    - c'est lui qui a créé votre utilisateur et déposé votre clé pour se co en SSH !
    - vous pouvez vérifier qu'il s'est bien déroulé avec la commande `cloud-init status`

> Pratique de pouvoir se connecter en utilisant une IP publique comme ça ! En revanche votre offre *Azure for Students* ne vous donne le droit d'utiliser que 3 IPs publiques. Pensez donc bien à supprimer les ressources au fur et à mesure du TP.

# II. Un ptit LAN

🌞 **Créez deux VMs depuis le Azure CLI**

- assurez-vous qu'elles ont une IP privée (avec `ip a`)
- elles peuvent se `ping` en utilisant cette IP privée
- deux VMs dans un LAN quoi !

> *N'hésitez pas à vous rendre sur la WebUI de Azure pour voir vos VMs créées.*

---

c