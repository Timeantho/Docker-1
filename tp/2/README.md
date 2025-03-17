# TP2 : spawn moar vm moar cloud much cloud : une approche programmatique

Dans ce deuxième TP, on va faire quasiment exactement la même chose qu'au TP1 maiiiiis en touchant de moins en moins à des boutons, et **avec de + en + de techniques de déploiement.**

Au menu :

- **création de VM depuis le shell `az`**
  - `az` comme Azure
  - un outil dév par Microsoft pour manipuler Azure depuis la ligne de commande
  - beaucoup plus pratique : on peut faire des scripts !
- **utilisation de `cloud-init`**
  - un outil pré-installé dans toutes les versions "cloud" des OS
  - genre y'a une version "cloud" de Ubuntu (que vous avez utilisé au TP1 déjà, que Azure met à dispo)
  - il s'exécute une seule fois : au premier boot de la machine
  - il configure des bailz, + de détails à la partie dédiée
- **création de VM avec Terraform**
  - un outil standard, qui fonctionne avec un peu tous les providers Cloud
  - tu écris dans un fichier texte ce que tu veux créer
  - tu exécutes `terraform apply` et *pouif* ça déploie tout
  - donc en un seul `terraform apply`, on peut déployer un parc entier si on veut :d


# Suite du TP

➜ **Vous allez avoir besoin de télécharger sur votre PC :**

- [le shell `az`](https://learn.microsoft.com/fr-fr/cli/azure/install-azure-cli)
- [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

➜ [**Part I** : Programmatic approach](part1.md)

➜ [**Part II** : cloud-init](part2.md)

➜ [**Part III** : Terraform](part3.md)
