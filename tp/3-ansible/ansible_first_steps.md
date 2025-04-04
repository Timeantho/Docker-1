# I. Premiers pas Ansible

Dans cette partie, je vous fait :

- setup une install d'Ansible sur votre PC
- on prépare les VMs
- **et on déploie de la conf !**

## Sommaire

- [I. Premiers pas Ansible](#i-premiers-pas-ansible)
  - [Sommaire](#sommaire)
  - [1. Mise en place](#1-mise-en-place)
    - [A. Setup Azure](#a-setup-azure)
    - [B. Setup sur votre poste](#b-setup-sur-votre-poste)
  - [2. La commande `ansible`](#2-la-commande-ansible)
  - [3. Un premier playbook](#3-un-premier-playbook)
  - [3. Création de nouveaux playbooks](#3-création-de-nouveaux-playbooks)
    - [A. NGINX](#a-nginx)
    - [B. MariaDB ou MySQL](#b-mariadb-ou-mysql)

## 1. Mise en place

### A. Setup Azure

➜ **Préparez un plan Terraform `main.tf`** (dans un nouveau répertoire de travail dédié à ce TP)

- il crée 2 VMs
- chacune doit être joignable en SSH depuis votre poste
- doit utiliser une image proposée par Azure un minimum à jour
- ajouter une conf `cloud-init.txt` :
  - Ansible et Python installés (référez-vous à la doc d'install de Ansible pour votre l'OS que vous avez choisi)
  - création d'un user qui a accès aux droits `root` avec la commande `sudo`
  - je vous conseille une conf en `NOPASSWD` pour ne pas avoir à saisir votre password à chaque déploiement
  - ce user a une clé publique pour vous y connecter sans mot de passe

🌞 **Vous me livrerez vos deux fichiers en compte-rendu**

```
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "chat_rg" {
  name     = "chat-rg"
  location = "East US"
}

resource "azurerm_virtual_network" "chat_vnet" {
  name                = "chat-vnet"
  address_space       = ["10.3.0.0/16"]
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
}

resource "azurerm_subnet" "chat_subnet" {
  name                 = "chat-subnet"
  resource_group_name  = azurerm_resource_group.chat_rg.name
  virtual_network_name = azurerm_virtual_network.chat_vnet.name
  address_prefixes     = ["10.3.1.0/24"]
}

resource "azurerm_network_security_group" "chat_nsg" {
  name                = "chat-nsg"
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
}

resource "azurerm_network_security_rule" "chat_ssh_rule" {
  name                        = "SSH-chat-rule"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.chat_nsg.name
}

resource "azurerm_public_ip" "chat1_public_ip" {
  name                = "chat1-public-ip"
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
  allocation_method   = "Dynamic"
}

resource "azurerm_public_ip" "chat2_public_ip" {
  name                = "chat2-public-ip"
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
  allocation_method   = "Dynamic"
}

resource "azurerm_network_interface" "chat1_nic" {
  name                = "chat1-nic"
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
  ip_configuration {
    name                          = "chat1-internal"
    subnet_id                     = azurerm_subnet.chat_subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.chat1_public_ip.id
  }
}

resource "azurerm_network_interface" "chat2_nic" {
  name                = "chat2-nic"
  location            = azurerm_resource_group.chat_rg.location
  resource_group_name = azurerm_resource_group.chat_rg.name
  ip_configuration {
    name                          = "chat2-internal"
    subnet_id                     = azurerm_subnet.chat_subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.chat2_public_ip.id
  }
}

resource "azurerm_linux_virtual_machine" "chat1_vm" {
  name                = "chat1-vm"
  resource_group_name = azurerm_resource_group.chat_rg.name
  location            = azurerm_resource_group.chat_rg.location
  size                = "Standard_B1s"
  admin_username      = "chatuser"
  network_interface_ids = [azurerm_network_interface.chat1_nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  custom_data = filebase64("${path.module}/cloud-init.txt")
}

resource "azurerm_linux_virtual_machine" "chat2_vm" {
  name                = "chat2-vm"
  resource_group_name = azurerm_resource_group.chat_rg.name
  location            = azurerm_resource_group.chat_rg.location
  size                = "Standard_B1s"
  admin_username      = "chatuser"
  network_interface_ids = [azurerm_network_interface.chat2_nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  custom_data = filebase64("${path.module}/cloud-init.txt")
}


cloud init.txt

#cloud-config
users:
  - name: chatuser
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    ssh_authorized_keys:
      - ssh-rsa AAAAB3... <your-public-key>

package_upgrade: true
packages:
  - python3
  - python3-pip
  - ansible

```

### B. Setup sur votre poste

➜ **Installez Ansible sur votre machine**

- il vous faudra aussi Python
- suivez [la doc officielle pour l'install](https://docs.ansible.com/ansible/latest/installation_guide/index.html)

➜ **Toujours dans le même répertoire de travail sur votre PC, créez un dossier `ansible/`**

- il accueillera le code Ansible pour ce TP

➜ **Préparez la connexion Ansible aux VMs**

- créez un fichier `ansible/.ssh-config` avec le contenu suivant

```ssh-config
Host 10.3.1.*
  User <VOTRE_USER>
  IdentityFile <VOTRE_CLE_PRIVEE>
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentitiesOnly yes
  LogLevel FATAL
```

- créez un fichier `ansible.cfg` avec le contenu suivant

```ini
[ssh_connection]
ssh_args = -F ./.ssh-config
```

> Si vous êtes pas trop bordéliques, vous avez donc actuellement dans votre répertoire de travail : `main.tf`, `cloud-init.txt` et le dossier `ansible/` qui contient le fichier `ansible/.ssh-config`.

## 2. La commande `ansible`

On va enfin utiliser un peu Ansible !

➜ Pour cela, **créez un fichier `ansible/hosts.ini`** avec le contenu suivant :

```ini
[tp3]
<IP DE LA PREMIERE VM>
<IP DE LA DEUXIEME VM>
```

On va commencer avec quelques commandes Ansible pour exécuter des tâches *ad-hoc* : c'est à dire des tâches one shot depuis la ligne de commandes.

```bash
$ cd ansible

# lister les hôtes que Ansible voit dans notre inventaire
$ ansible -i hosts.ini tp3 --list-hosts

# tester si ansible est capable d'interagir avec les machines
$ ansible -i hosts.ini tp3 -m ping

# afficher toutes les infos que Ansible est capable de récupérer sur chaque machine
$ ansible -i hosts.ini tp3 -m setup

# exécuter une commande sur les machines distantes
$ ansible -i hosts.ini tp3 -m command -a 'uptime'

# exécuter une commande en root
$ ansible -i hosts.ini tp3 --become -m command -a 'reboot'
```

## 3. Un premier playbook

Enfin, on va écrire un peu de code Ansible.  
On va commencer simple et faire un *playbook* en un seul fichier, pour prendre la main sur la syntaxe, et faire un premier déploiement.

➜ **créez un fichier `first.yml`**, notre premier *playbook* :

```yaml
---
- name: Install nginx
  hosts: tp3
  become: true

  tasks:
  - name: Install nginx
    dnf:
      name: nginx
      state: present

  - name: Insert Index Page
    template:
      src: index.html.j2
      dest: /usr/share/nginx/html/index.html

  - name: Start NGiNX
    service:
      name: nginx
      state: started
```

> Chaque élément de cette liste YAML est donc une *task* Ansible. Les mots-clés `yum`, `template`, `service` sont des *modules* Ansible.

➜ **Et un fichier `index.html.j2` dans le même dossier**

```jinja2
Hello from {{ ansible_default_ipv4.address }}
```

➜ **Exécutez le playbook**

```bash
$ ansible-playbook -i hosts.ini first.yml
```

> N'oubliez pas d'ouvrir le port 80 avec une règle Azure (modifiez et ré-appliquez votre fichier `main.tf`).

## 3. Création de nouveaux playbooks

### A. NGINX

➜ **Créez un *playbook* `nginx.yml`**

- déploie un serveur NGINX
- génèreérer un certificat et une clé au préalable
  - le certificat doit être déposé dans `/etc/pki/tls/certs`
  - la clé doit être déposée dans `/etc/pki/tls/private`
- créer une racine web et un index
  - créez le dossier `/var/www/tp3_site/`
  - créez un fichier à l'intérieur `index.html` avec un contenu de test
- déploie une nouveau fichier de conf NGINX
  - pour servir votre `index.html` en HTTPS (port 443)
- ouvre le port 443/TCP dans le firewall

➜ **Modifiez votre `hosts.ini`**

- ajoutez une section `web`
- elle ne contient que `10.3.1.11`

➜ **Lancez votre playbook sur le groupe `web`**

➜ **Vérifiez que vous accéder au site avec notre navigateur**

> S'il y a besoin d'aide pour tout ça, si vous n'êtes pas du tout familier avec la conf NGINX, n'hésitez pas à faire appel à moi.

🌞 **Pour le compte-rendu**

```
hosts.ini

[tp3]
10.3.1.11
10.3.1.12

[web]
10.3.1.11

```

> N'oubliez pas d'ouvrir le port 443 avec une règle Azure (modifiez et ré-appliquez votre fichier `main.tf`).

### B. MariaDB ou MySQL

➜ **Créez un *playbook* `mariadb.yml`** (ou `mysql.yml`)

- déploie un serveur 
  - MariaDB si vous avez choisi une base RedHat (Rocky, Alma, etc.)
  - ou MySQL si vous utilisez une base Debian  (Debian, Ubuntu, etc.)
- crée un user SQL ainsi qu'une base de données sur laquelle il a tous les droits

➜ **Modifiez votre `hosts.ini`**

- ajoutez une section `db`
- elle ne contient que `10.3.1.12`

➜ **Lancez votre playbook sur le groupe `db`**

➜ **Vérifiez en vous connectant à la base que votre conf a pris effet**

🌞 **Pour le compte-rendu**

- `mariadb.yml` (ou `mysql.yml`) et `hosts.ini` dans le compte-rendu

```
mariadb.yml (code copié collé depuis le site officiel de mariadb)

---
- name: Install and configure MariaDB
  hosts: db
  become: true

  tasks:
    - name: Install MariaDB server
      dnf:
        name: mariadb-server
        state: present

    - name: Enable and start MariaDB service
      service:
        name: mariadb
        state: started
        enabled: true

    - name: Create a database for chat app
      mysql_db:
        name: chat_db
        state: present

    - name: Create a user and grant all privileges on chat_db
      mysql_user:
        name: chat_user
        password: chat_password123
        priv: 'chat_db.*:ALL'
        state: present

    - name: Open port 3306 in the firewall
      firewalld:
        port: 3306/tcp
        permanent: true
        state: enabled


hosts.ini
[tp3]
10.3.1.11
10.3.1.12

[web]
10.3.1.11

[db]
10.3.1.12

```