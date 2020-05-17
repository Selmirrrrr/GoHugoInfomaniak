---
layout: post
title: Hugo x GitHub x Infomaniak
subtitle: Publier son Hugo blog sur Infomaniak avec GitHub Actions
tags: [blog, ownyourdata, cicd]
date: 2020-05-05T08:10:26+08:00
lastmod: 2020-05-05T08:10:26+08:00
comments: true
draft: true

---
Comme tout bon développeur qui se respecte, je passe malheuresement beaucoup plus de temps à tester différents outils pour mon blog qu'à réellement écrire du contenu pour l'alimenter.

Pour poursuivre dans cette lignée j'ai cette fois décidé de migrer mon blog de [Jekyll](https://jekyllrb.com/) à [Hugo](https://gohugo.io) (pourquoi pas) et surtout de changer l'hébergeur en passant de [GitHub Pages](https://pages.github.com/) à notre bon ~~vieux~~ [Infomaniak](https://infomaniak.ch) local :CH:.

Mais cette fois, pour remédier à me flémingite par rapport à la création de contenus, j'ai décidé de me baser sur cette expérience de migration pour en faire un petit article afin de vous montrer comment, très facilement, vous pouvez vous aussi créer votre blog avec [Hugo](https://gohugo.io) et l'héberger sur [Infomaniak](https://infomaniak.ch).

{{< admonition warning "Attention" true >}}
Ce blog s'adressant en grande partie à un public de développeurs, je passerai un peu rapidement sur certains aspects techniques non-essentiels, mais n'hésitez pas à poser vos questions en commentaire, j'y répondrai avec grand plaisir.
{{< /admonition >}}

## 1. Pré-requis
 - Compte hébérgement mutualisé Infomaniak
 - [Un site Hugo en local](https://gohugo.io/getting-started/quick-start/)
 - Compte GitHub pour le déploiement continu

## 2. Infomaniak
### 2.1 Compte FTP
Pour commencer, rendez-vous dans votre console d'administration de votre hébergement Infomaniak pour le création de votre compte SFTP qui vous permettra de déplyoer votre site sur les serveurs d'Infomaniak.

 1. [Console Infomaniak](https://manager.infomaniak.com/v3)
 2. Cliquer sur le button `Hébergement Web`
 3. Sélectionner votre hébergement dans la liste affichée
 4. Puis sur `FTP/SSH` dans la colonne de gauche
 5. Dans la section `Compte FTP` cliquer sur `Ajouter`
 6. Pour finir, renseigner les informations demandées et cliquer sur `Valider`

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/sftp-infomaniak.png" caption="Création compte FTP Infomaniak" >}}

## 3. GitHub
### 3.1 Créer un repository
Pour les dévloppeurs agueris à GitHub cet étape est triviale, néanmoins, si besoin voici la [marche à suivre](https://help.github.com/en/github/getting-started-with-github/set-up-git) officielle.

### 3.2 Secrets
Pour que GitHub puisse déployer le site sur l'hébergement il faut lui fournir les informations de connexion au compte (S)FTP précdemaent crée.

{{< admonition danger "Danger" true >}}
Ne jamais, **JAMAIS**, directement écrire vos informations d'accès directement dans les fichiers qui seront directement publiés sur votre répo/blog.
{{< /admonition >}}

Pour ceci, nous allons utiliser `GitHub Secrets` qui va s'occuper de stocker nos informations de connexion de façon sécurisée.

 1. Dans l'onglet `Settings` de votre répo
 2. Cliquer sur le menu `Secrets`
 3. Puis ajouter les "secrets" suivants : 
    - FTP_USER
      - le nom d'utilisateur du compte FTP crée précédamment
    - FTP_PASSWORD
      - le mot de passe du compte FTP crée précédamment
    - FTP_SERVER
      - l'adresse de votre serveur FTO (générallement ftp.votredomaine.ch)

<br />

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/github-secrets.png" caption="GitHub Secrets" >}}

### 3.3 Action
Pour créer un workflow `GitHub Action` il faut simplement créer un fichier `YAML` dans le lquel il faut décrire les différentes opérations à effectuer pour déployer notre site.

  1. À la racine du dossier contenant le site Hugo créer un dossier `.github`
  2. Dans ce dossier créer un dossier `workflows`
  3. Puis dans `workflows` créer un fichier `hugo2infomaniak.yml` (nommez le fichier comme vous voulez) puis ouvrez les dans votre éditeur de texte préféré.

### 3.4 Flow
Ce qu'il nous faut ici c'est un workflow qui va pousser chacune de nos modifications dans Infomaniak via GitHub.

Schématiquement voici le résultat souhaité : 

{{< mermaid >}}
graph LR;
    A[Nouveau contenu] -->|git push| B(Génération du blog)
    B -->|hugo --minify| C{Build réussi}
    C -->|Oui| D(Déployer)
    C -->|Non| E(Erreur)
{{< /mermaid >}}

Dans le fichier `hugo2infomaniak.yml` précédament crée intégrer le contenu suivant :

```yaml
name: hugo2infomaniak

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
```

## 4. Hugo
### 4.1 Fichiers
Il ne nous reste maintenant plus qu'à définir quels fichiers seront envoyés sur GitHub et sur Infomaniak.

Pour ceci, il vous souffit de télécharger les deux fichiers suivants et de les placer à racine du dossier contenant votre blog : 

 - [.gitignore](/2020/05/Hugo-Infomaniak-GithubActions/.gitignore)  
  Ce fichier va "dire" à Git de ne pas versionner certains fichiers (généralement les fichiers auto générés). Voir [ici](https://openclassrooms.com/fr/courses/2342361-gerez-votre-code-avec-git-et-github/2433721-ignorez-des-fichiers) pour plus d'infos.
 - [.git-ftp-include](/2020/05/Hugo-Infomaniak-GithubActions/.git-ftp-include)  
   À l'inverse, ce fichier va dire à [git-ftp](https://github.com/git-ftp/git-ftp) quels fichiers envoyer à notre hébérgement, ici en l'occurence tout le contenu du dossier `public` dans le quel Hugo va générer tout le contenu du blog.

### 4.2 Arborescence
À la fin de toutes ces étapes vous devriez une arboresecence en local qui ressemble à ça : 

```
.
│   .git-ftp-include
│   .gitignore
│   config.toml
|
├───.github/
│   └───workflows/
│           hugo2sftp.yml
│
├───archetypes/
│       default.md
│
├───content
│   └───posts
│           my-first-post.md
|
├───data/
├───layouts/
├───static/
└───themes/
```

# It's magic
