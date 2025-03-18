# Part III : Terraform

**Dans cette dernière partie on va explorer une utilisation basique de Terraform.**

On utilisera Terraform pour automatiser la création de machines dans Azure, *via* des fichiers texte avec la syntaxe Terraform.

> Ouais, encore une new syntaxe ! Bah bienvenue dans le monde du cloud/devops toussa : on est des admins qui écrivons du code (Dockerfile, docker-compose.yml, Terraform, et d'autres) pour déployer des machines et de la conf !

![APPLY](./img/apply.jpg)

Terraform va permettre d'automatiser la création de ressources dans Azure, *Resource Group* comme VMs, ou tout autre type de ressource que sait gérer Azure.

➜ Je vous laisse là encore suivre **[la documentation officielle de Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)** pour l'installer sur votre poste

# Sommaire

- [Part III : Terraform](#part-iii--terraform)
- [Sommaire](#sommaire)
  - [1. Introooo](#1-introooo)
  - [2. Copy paste](#2-copy-paste)
  - [3. Do it yourself](#3-do-it-yourself)
  - [4. cloud-iniiiiiiiiiiiiit](#4-cloud-iniiiiiiiiiiiiit)

## 1. Introooo

Les fichiers Terraform pourtent l'extension `.tf` et la syntaxe utilisée est appelée HCL.

> Une énième syntaxe pour simplement déclarer des clés et des valeurs :)

**On appelle *Plan* un fichier Terraform qui contient des ressources à créer, à l'aide d'un *Provider* donné.**

Dans notre cas, on utilisera le provider `azurerm`.

> Il vous faudra repérer le "subcription ID" de votre Azure for Student. Vous pouvez le voir en saisissant "subscription" dans la barre de recherche sur l'interface Web de Azure.

Voici le minimum requis, [recommandé dans les exemples de la doc](https://github.com/hashicorp/terraform-provider-azurerm/blob/main/website/docs/r/linux_virtual_machine.html.markdown), pour créer une VM avec Azure en provider :

> En vrai ils marchaient pas ouf ou il manquait des trucs, j'vous ai mâché le travail parce que c'est un enfer Azure hihi.

```hcl
provider "azurerm" {
  features {}
  subscription_id = "<TON_SUBSCRIPTION_ID>"
}
resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}
resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}
resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "<TON_USERNAME>"
  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "<TON_USERNAME>"
    public_key = file("<CHEMIN_VERS_TA_CLE_PUBLIQUE>")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
```

➜ **Remarquez dans fichier plusieurs choses**

- **le bloc `terraform {}` tout en haut du doc**
  - nécessaire
  - définit notamment le *provider* nécessaire pour que l'on puisse appliquer ce *plan*
- **le bloc `provider "azurerm" {}`**
  - nécessaire, même si non-utilisé (comme ici)
- **les blocs `resource`**
  - sont les ressources que l'on souhaite créer
  - elles sont spécifiques au *provider* choisi
    - par exemple, le nom `azurerm_linux_virtual_machine` est spécifique au *provider* `azurerm`
- **utilisation de variables**
  - ouais y'a un deuxième fichier qui doit être créé juste à côté en fait, `variables.tf` :

```hcl
variable "prefix" {
  description = "da prefix"
  default = "tp2magueule"
}
variable "location" {
  description = "da location"
  default = "West Europe"
}
```


## 2. Copy paste

> *Pour rappel, Terraform est à utiliser depuis votre poste. Les fichiers à créer sont donc aussi à créer sur votre poste.*

➜ **Créer un *plan* Terraform**

- créer un nouveau répertoire de travail (un dossier vide quoi, pour pas foutre le bordel j'sais pas où :D)
- créer un fichier `main.tf`
  - dans le répertoire de travail
  - remplissez-le avec le fichier d'exemple présenté au dessus
  - remplacez les machins à remplacer :
    - `<TON_USERNAME>`
    - `<CHEMIN_VERS_TA_CLE_PUBLIQUE>`
    - `<TON_SUBSCRIPTION_ID>`
- créer aussi un fichier `variables.tf` juste à côté
  - pareil, avec le contenu que je vous ai filé au dessus

➜ **Depuis un shell, appliquer le plan Terraform**

- depuis un shell, se déplacer dans le répertoire de travail
- exécuter les commandes suivantes :

```bash
# Récupération du provider azurerm
$ terraform init

# Vérification de la validité du plan
$ terraform plan

# Déploiement du plan
$ terraform apply
```

🌞 **Constater le déploiement**

- depuis la WebUI si tu veux
- pour le compte-rendu : depuis le CLI `az`
  - `az vm list`
  - `az vm show --name VM_NAME --resource-group RESOURCE_GROUP_NAME`
  - `az group list`
  - n'oubliez pas que vous pouvez ajouter `-o table` pour avoir un output plus lisible par un humain :)

➜ **Autres commandes Terraform**

```bash
# Vérifier que votre fichier .tf est valide
$ terraform validate

# Formate un fichier .tf au format standard
$ terraform fmt

# Afficher les ressources du déploiement
$ terraform state list

# Afficher les détails d'une des ressources du déploiement
$ terraform state show <RESSOURCE>

# Détruit les ressources déployées
$ terraform destroy
```

## 3. Do it yourself

🌞 **Créer un *plan Terraform* avec les contraintes suivantes**

- `node1`
  - Ubuntu 22.04
  - 1 IP Publique
  - 1 IP Privée
- `node2`
  - Ubuntu 22.04
  - 1 IP Privée
- les IPs privées doivent permettre aux deux machines de se `ping`

> Pour accéder à `node2`, il faut donc d'abord se connecter à `node1`, et effectuer une connexion SSH vers `node2`. Vous pouvez ajouter l'option `-j` de SSH pour faire ~~des dingueries~~ un rebond SSH (`-j` comme Jump). `ssh -j node1 node2` vous connectera à `node2` en passant par `node1`.

## 4. cloud-iniiiiiiiiiiiiit

🌞 **Intégrer la gestion de `cloud-init`**

- faire pop une VM Ubuntu 22.04 qui utilise `cloud-init` au premier boot
- ce `cloud-init` doit faire pareil qu'à la partie précédente :
  - installer Docker sur la machine
  - ajoutez un user qui porte votre pseudo
    - il a un password défini
    - clé SSH publique déposée
    - membre du groupe `docker`
  - l'image Docker `alpine:latest` doit être téléchargée

🌞 **Moar `cloud-init` and Terraform configuration**

- adaptez le plan Terraform et le fichier `cloud-init` précédents
- un `docker-compose.yml` est automatiquement déposé
  - il se situe dans `/opt/wikijs`
  - c'est le `docker-compose.yml` de la doc officielle de WikiJS (pareil qu'au TP1)
- le `docker-compose.yml` est automatiquement démarré
- par défaut, WikiJS écoute sur le port 3000
  - vous devrez modifier le `docker-compose.yml` pour que ce port soit partagé sur le port 10101 de la machine hôte
- le Network Security Group associé à l'interface autorise le traffic sur le port 10101 spécifiquement

🌞 **Play with compose**

- ajoutez dans le `docker-compose.yml` un troisième conteneur NGINX
  - agit comme un reverse proxy pour accéder au WikiJS
  - uniquement une connexion chiffrée avec TLS (`https://` quoi)
    - générez un certificat auto-signé pour ça
  - il écoute sur le port 443 dans le conteneur (la conf NGINX)
  - il est dispo sur le port 443 de l'hôte (partage de port Docker)
- ce conteneur reverse-proxy est le **seul** moyen d'accéder à WikiJS
  - il faut enlever le partage de port du conteneur WikiJS
- ajouter des `healthcheck` dans le `docker-compose.yml`
  - une mécanique pour qu'un conteneur soit déterminé comme en bonne santé quand il valide une commande
  - le conteneur NGINX doit être marqué comme *healthy* quand son port 443 est joignable
  - le conteneur WikiJS doit être marqué comme *healthy* quand son port 3000 est joignable
- ajouter un `depends_on` pour forcer NGINX à démarrer après WikiJS

![Terraforming Mars](./img/terraforming_mars.jpg)
