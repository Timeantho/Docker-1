# II.3. Setup réseau

![VXLAN](./img/vxlan_bornt.png)

**Ici on va préparer le réseau virtuel VXLAN que les VMs vont utiliser pour communiquer entre elles.**

Grâce à VXLAN, des VMs qui se situent sur des hyperviseurs différents pourront communiquer comme si elles étaient dans le même réseau local.

## Sommaire

- [II.3. Setup réseau](#ii3-setup-réseau)
  - [Sommaire](#sommaire)
  - [A. Intro](#a-intro)
  - [B. Création du Virtual Network](#b-création-du-virtual-network)
  - [C. Préparer le bridge réseau](#c-préparer-le-bridge-réseau)

## A. Intro

Pour le réseau, la plupart des étapes sont gérées par OpenNebula, notamment la création des endpoints VXLAN.

Il restera quelques étapes manuelles à effectuer, afin de monter un setup minimaliste avec des fonctionnalités attendues pour de la virtu :

- pouvoir contacter les VMs de l'extérieur, pour pouvoir s'y co en SSH par exemple
- avoir un réseau local entre les VMs pour qu'elles se joignent entre elles
- proposer un accès internet aux VMs

## B. Création du Virtual Network

➜ **RDV de nouveau sur la WebUI de OpenNebula, et naviguez dans `Network > Virtual Networks`**

➜ **Créer un nouveau Virtual Network, et renseignez :**

- un nom de réseau (ce que vous voulez)
- le mode VXLAN
- Onglet `Conf`
  - spécifiez en interface réseau physique l'interface qui a une IP statique sur votre machine (genre `eth0` ou `enp0s3` ou autre chose, j'peux pas deviner :d)
  - définissez `vxlan_bridge` en nom de bridge (attention aux commandes plus bas si vous choisissez un autre nom)
- Onglet `Addresses`
  - spécifiez une IP de départ dans `First IPv4 address`, par exemple `10.220.220.1`
  - indiquez aussi ainsi qu'un nombre de machines possibles dans ce réseau avec `Size` (peu importe pour nos tests, mettez genre 50)
- Onglet `Context`
  - spécifier une adresse de réseau et un masque dans les champs dédiés, par exemple `10.220.220.0` et `255.255.255.0`

## C. Préparer le bridge réseau

➜ **Ces étapes sont à effectuer uniquement sur `kvm1.one`** dans un premier temps

- dans la partie IV du TP, quand vous mettrez en place `kvm2.one`, il faudra aussi refaire ça dessus

🌞 **Créer et configurer le bridge Linux**, j'vous file tout, suivez le guide :

```bash
# création du bridge
ip link add name vxlan_bridge type bridge

# on allume le bridge
ip link set dev vxlan_bridge up 

# on définit une IP sur cette interface bridge
ip addr add 10.220.220.201/24 dev vxlan_bridge

# ajout de l'interface bridge à la zone public de firewalld
firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent

# activation du masquerading NAT dans cette zone
firewall-cmd --add-masquerade --permanent

# on reload le firewall pour que les deux commandes précédentes prennent effet
firewall-cmd --reload
```

➜ Je vous conseille **très fort** de faire en sorte que ce script s'exécute automatiquement au démarrage. 

En faisant un ptit service systemd par exemple, que vous pouvez `enable` ensuite :

```bash
# on suppose que le script est dans /opt/vxlan.sh
$ ls /opt
vxlan.sh

$ cat /etc/systemd/system/vxlan.service
[Unit]
Description=Setup VXLAN interface for ONE

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/bash /opt/vxlan.sh

[Install]
WantedBy=multi-user.target

# quand vous créez ce fichier, ordonnez à systemd de le lire
$ sudo systemcl daemon-reload

# puis on peut le manipuler avec systemctl
$ sudo systemctl start vxlan
$ sudo systemctl enable vxlan
```