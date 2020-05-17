---
layout: post
title: Hugo x Infomaniak x GitHub
subtitle: Publier son Hugo blog sur Infomaniak avec GitHub Actions
tags: [blog, ownyourdata, cicd]
date: 2020-05-05T08:10:26+08:00
lastmod: 2020-05-05T08:10:26+08:00
comments: true
draft: true

---
Comme tout bon développeur qui se respecte, je passe beaucoup plus de temps à tester différents outils pour mon blog qu'à réellement écrire du contenu pour l'alimenter.

Pour poursuivre dans cette lignée j'ai cette fois décidé de migrer mon blog de [Jekyll](https://jekyllrb.com/) à [Hugo](https://gohugo.io) (pourquoi pas) et surtout de changer l'hébergeur en passant de [GitHub Pages](https://pages.github.com/) à notre bon ~~vieux~~ [Infomaniak](https://infomaniak.ch) local :CH:.

Mais cette fois, pour remédier à me flémingite par rapport à la création de contenus, j'ai décidé de me baser sur cette expérience de migration pour en faire un petit article afin de vous montrer comment, très facilement, vous pouvez vous aussi créer votre blog avec [Hugo](https://gohugo.io) et l'héberger sur [Infomaniak](https://infomaniak.ch).

## Pré-requis
 - Compte avec hébérgement mutualisé Infomaniak
 - [Un site Hugo en local](https://gohugo.io/getting-started/quick-start/)
 - Compte GitHub pour le déploiement continu

## Infomaniak
### Création compte SFTP
Pour commencer, rendez-vous dans votre console d'administration de votre hébergement Infomaniak pour le création de votre compte SFTP qui vous permettra de déplyoer votre site sur les serveurs d'Infomaniak.

 1. [Console Infomaniak](https://manager.infomaniak.com/v3)
 2. Cliquer sur le button `Hébergement Web`
 3. Sélectionner votre hébergement dans la liste affichée
 4. Puis sur `FTP/SSH` dans la colonne de gauche
 5. Dans la section `Compte FTP` cliquer sur `Ajouter`
 6. Pour finir, renseigner les informations demandées et cliquer sur `Valider`

![création compte sftp infomaniak](/2020/05/Hugo-Infomaniak-GithubActions/sftp-infomaniak.png)

### Attention compte FTP
{{< admonition warning "Attention" true >}}
Dans le cas d'un hébergement Infomaniak gratuit, il est impossible créer de compte SFTP mais un compte FTP oui.
{{< /admonition >}}

Je comprends qu'Infomaniak doit susciter le besoin de passer aux offres payantes, mais en 2020 ne pas proposer de base un accès SFTP me sidère. Mais, ainsi soit-il, cela ne nous empêchera pas de poursuivre cette marche à suivre et d'héberger notre blog chez eux.

Il faut juste être conscient que, dans le cas d'un compte FTP, les accès transiteront en clair entre GitHub et Infomaniak. Le risque de compromission est très faible mais il existe et il faut le savoir.

## GitHub
### Création d'un nouveau repository
Pour les dévloppeurs agueris à GitHub cet étape est triviale, néanmoins, si besoin voici la [marche à suivre](https://help.github.com/en/github/getting-started-with-github/set-up-git) officielle.

### GitHub Secrets
Pour que GitHub puisse déployer le site sur l'hébergement il faut lui fournir les informations de connexion au compte (S)FTP précdemaent crée.

 1. Dans l'onglet `Settings` de votre répo
 2. Cliquer sur le menu `Secrets`
 3. Puis ajouter les "secrets" suivants : 
    - FTP_USER
      - le nom d'utilisateur du compte FTP crée précédamment
    - FTP_PASSWORD
      - le mot de passe du compte FTP crée précédamment
    - FTP_SERVER
      - l'adresse de votre serveur FTO (générallement ftp.votredomaine.ch)

## Hugo
Une fois le site Hugo prêt et fonctionnel en local nous pouvons commencer à mettre les 2-3 petites choses nécessaires à GitHub Actions pour qu'il puisse déployer notre site sur notre hébergement.

### Action
Pour créer un workflow `GitHub Action` il faut simplement créer un fichier `YAML` dans le lquel il faut décrire les différentes opérations à effectuer pour déployer notre site.

### Création
  1. À la racine du dossier contenant le site Hugo créer un dossier `.github`
  2. Dans ce dossier créer un dossier `workflows`
  3. Puis dans `workflows` créer un fichier `hugo2infomaniak.yml` (nommez le fichier comme vous voulez) puis ouvrez les dans votre éditeur de texte préféré.

### Flow
Ce qu'il nous faut ici c'est un workflow qui va pousser chacune de nos modifications dans Infomaniak via GitHub.

Schématiquement voici le résultat souhaité : 

{{< mermaid >}}
graph LR;
    A[Nouveau contenu dans Hugo] -->|git push| (Génération du blog)
    B -->|hugo --minify| C{Build réussi}
    C -->|Oui| [Déployer sur (S)FTP]
    C -->|Non| [Erreur de génération]
{{< /mermaid >}}

```yaml
name: hugo2ftp

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
        
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true 

      - name: Build
        run: hugo --minify
      
      - name: SFTP Deploy
        uses: SamKirkland/FTP-Deploy-Action@3.0.0
        with:
          ftp-server: ${{ secrets.FTP_SERVER }}
          ftp-username: ${{ secrets.FTP_USER }}
          ftp-password: ${{ secrets.FTP_PASSWORD }}
          local-dir: public/
          git-ftp-args: --insecure
```