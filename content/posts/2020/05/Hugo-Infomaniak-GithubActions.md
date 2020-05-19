---
layout: post
title: Hugo x GitHub x Infomaniak
subtitle: Publier son blog Hugo sur Infomaniak avec GitHub Actions
tags: [blog, ownyourdata, cicd]
date: 2020-05-05T08:10:26+08:00
lastmod: 2020-05-17T18:10:26+08:00
comments: true
draft: false

---
Comme tout bon développeur qui se respecte, je passe malheureusement beaucoup plus de temps à tester différents outils pour mon blog qu'à réellement écrire du contenu pour l'alimenter.

Pour poursuivre dans cette lignée, j'ai cette fois décidé de migrer mon blog de [Jekyll](https://jekyllrb.com/) à [Hugo](https://gohugo.io) (pourquoi pas) et surtout de changer l'hébergeur en passant de [GitHub Pages](https://pages.github.com/) à notre bon ~~vieux~~ [Infomaniak](https://infomaniak.ch) local.

Mais cette fois, pour remédier à me flemmingite par rapport à la création de contenus, j'ai décidé de me baser sur cette expérience de migration pour en faire un petit article afin de vous montrer comment, très facilement, vous pouvez vous aussi créer votre blog avec [Hugo](https://gohugo.io) et l'héberger chez [Infomaniak](https://infomaniak.ch).

{{< admonition warning "Attention" true >}}
Ce blog s'adressant en grande partie à un public de développeurs, je passerai un peu rapidement sur certains aspects techniques non-essentiels, mais n'hésitez pas à poser vos questions en commentaire, j'y répondrai avec grand plaisir.
{{< /admonition >}}

## 1. Prérequis
 - Compte hébergement mutualisé [Infomaniak](https://infomaniak.ch)
 - [Un site Hugo en local](https://gohugo.io/getting-started/quick-start/)
 - Compte [GitHub](https://github.com)

## 2. Infomaniak
### 2.1 Compte FTP
Pour commencer, rendez-vous dans votre console d'administration de votre hébergement Infomaniak pour la création de votre compte SFTP qui vous permettra de déployer votre site sur leurs serveurs.

 1. [Console Infomaniak](https://manager.infomaniak.com/v3)
 2. Cliquer sur le bouton `Hébergement Web`
 3. Sélectionner votre hébergement dans la liste affichée
 4. Puis sur `FTP/SSH` dans la colonne de gauche
 5. Dans la section `Compte FTP` cliquer sur `Ajouter`
 6. Pour finir, renseigner les informations demandées et cliquer sur `Valider`

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/sftp-infomaniak.png" caption="Création compte FTP Infomaniak" >}}

## 3. Git
### 3.1 Installation
Cet étape est probablement déjà faite pour 99% des lecteurs de ce blog c'est pourquoi je ne vais pas la détailler, je vous recommande simplement de suivre la [documentation](https://help.github.com/en/github/getting-started-with-github/set-up-git) de GitHub pour cela.

### 3.2 GitHub Repo
Pour créer un nouveau repository GitHub, se rendre sur [la page de création](https://github.com/new) et saisir les informations souhaitées : 

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/github-new-repo.png" caption="Création repo GitHub" >}}

Seul élément obligatoire, le nom du repo. Important également de choisir si vous voulez que votre repo soit privé ou public (visible à tous).

Le reste des paramètres peuvent être laissés à leur valeur par défaut.

### 3.3 GitHub Secrets
Pour que GitHub puisse déployer le site sur l'hébergement il faut lui fournir les informations de connexion au compte (S)FTP précédemment crée.

{{< admonition danger "Danger" true >}}
Ne jamais, **JAMAIS**, directement écrire d'informations d'accès directement dans les fichiers qui seront directement publiés sur le repo/blog.
{{< /admonition >}}

Pour ceci, nous allons utiliser `GitHub Secrets` qui va s'occuper de stocker nos informations de connexion de façon sécurisée.

 1. Allez dans l'onglet `Settings` de votre repo
 2. Cliquez sur le menu `Secrets`
 3. Puis ajoutez les "secrets" suivants : 
    - FTP_USER
      - le nom d'utilisateur du compte FTP crée précédemment
    - FTP_PASSWORD
      - le mot de passe du compte FTP crée précédemment
    - FTP_SERVER
      - l'adresse de votre serveur FTP (généralement ftp.votredomaine.ch) précédé de `ftpes://`

<br />

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/github-secrets.png" caption="GitHub Secrets" >}}

### 3.4 GitHub Action
Pour créer un workflow `GitHub Action`, il faut simplement créer un fichier `YAML` dans le lequel il faut décrire les différentes opérations à effectuer pour déployer notre site.

  1. À la racine du dossier contenant le site Hugo, créez un dossier `.github`
  2. Dans ce dossier, créez un dossier `workflows`
  3. Puis dans `workflows`, créez un fichier `hugo2infomaniak.yml` (nommez le fichier comme vous voulez) puis ouvrez les dans votre éditeur de texte préféré.

### 3.5 Workflow
Ce qu'il nous faut ici, c'est un workflow qui va pousser chacune de nos modifications dans Infomaniak via GitHub.

Schématiquement, voici le résultat souhaité : 

{{< mermaid >}}
graph LR;
    A[Nouveau contenu] -->|git push| B(Génération du blog)
    B -->|hugo --minify| C{Build réussi}
    C -->|Oui| D(Déployer)
    C -->|Non| E(Erreur)
{{< /mermaid >}}

Dans le fichier `hugo2infomaniak.yml` précédemment crée, intégrez le contenu suivant :

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

Pour ceci, il vous suffit de télécharger les deux fichiers suivants et de les placer à racine du dossier contenant votre blog : 

 - [.gitignore](/2020/05/Hugo-Infomaniak-GithubActions/.gitignore)  
  Ce fichier va "dire" à Git de ne pas versionner certains fichiers (généralement les fichiers auto générés). Voir [ici](https://openclassrooms.com/fr/courses/2342361-gerez-votre-code-avec-git-et-github/2433721-ignorez-des-fichiers) pour plus d'infos.
 - [.git-ftp-include](/2020/05/Hugo-Infomaniak-GithubActions/.git-ftp-include)  
   À l'inverse, ce fichier va dire à [git-ftp](https://github.com/git-ftp/git-ftp) quels fichiers envoyer à notre hébergement, ici en l'occurrence tout le contenu du dossier `public` dans le quel Hugo va générer le contenu du blog.

### 4.2 Arborescence
À la fin de toutes ces étapes l'arborescence en local devrait ressembler à ça : 

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

## 5. It's magic 🚀
Maintenant que tout cela est fait, il ne reste plus qu'à publier les changements effectués.

Pour ce faire, sur GitHub, sur la page du repo précédemment créée, récupérez l'URL du repo : 

{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/github-quick-setup.png" caption="GitHub - Quick setup page" >}}

Puis exécutez les commandes suivantes dans votre invite de commande :

```
$ git init
$ git commit -am "first commit"
$ git remote add origin URL_DU_REPO
$ git push
```

Dès que le commit sera arrivé sur GitHub, GitHub Actions prendra le relai pour builder le site et le publier chez Infomaniak.  
<br />
{{< image src="/2020/05/Hugo-Infomaniak-GithubActions/action-magic.png" caption="GitHub Action Magic" >}}

Après quelques minutes, il devrait être disponible sur l'URL de l'hébergement Infomaniak.

Pour les prochaines fois (après chaque articles ou modification), il suffira d'effecteur les commandes suivantes : 

```
$ git commit -am "mon super article"
$ git push
```

## 6. Conclusion
On devrait maintenant avoir le blog publié et fonctionnel sur Infomaniak (comme le blog que vous lisez en ce moment).

J'espère que l'article vous aura été utile, ce fût très compliqué pour moi de choisir sous quel angle "attaquer" cette marche à suivre car selon si vous avez un "background" informatique (voir développeur) l'article aurait pu faire 10 lignes comme il aurait pu en faire 1000 si on partait vraiment de 0. 

J'ai choisi l'entre-deux mais n'hésitez vraiment pas à me poser vos questions ou à demander de l'aide. Je reste très volontiers disponible pour cela via commentaire ici ou via Twitter.
