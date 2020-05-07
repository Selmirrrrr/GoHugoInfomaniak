---
layout: post
title: Hugo x Infomaniak x GitHub
subtitle: Publier son Hugo blog sur Infomaniak avec GitHub Actions
tags: [blog, ownyourdata, cicd]
date: 2020-05-05T08:10:26+08:00
lastmod: 2020-05-05T08:10:26+08:00
comments: true
featuredImage: "/images/gitbanner.png"

---
Comme tout bon développeur qui se respecte, je passe beaucoup plus de temps à tester différents outils pour mon blog qu'à réellement écrire du contenu pour l'alimenter.

Pour poursuivre dans cette lignée j'ai cette fois décidé de migrer mon blog de [Jekyll](https://jekyllrb.com/) à [Hugo](https://gohugo.io) (pourquoi pas) et surtout de changer l'hébergeur en passant de [GitHub Pages](https://pages.github.com/) à notre bon ~~vieux~~ [Infomaniak](https://infomaniak.ch) local :CH:.

Mais cette fois, pour remédier à me flémingite par rapport à la création de contenus, j'ai décidé de me baser sur cette expérience de migration pour en faire un petit article afin de vous montrer comment, très facilement, vous pouvez vous aussi créer votre blog avec [Hugo](https://gohugo.io) et l'héberger sur [Infomaniak](https://infomaniak.ch).

## Pré-requis
 - Hébergement mutualisé Infomaniak
 - Utilisateur (S)FTP Infomaniak
 - Facultatif : Compte GitHub pour le déploiement continu

## Création du blog
Pour cette première partie je ne vais pas réinviter la roue, si vous n'avez pas déjà un blog Hugo je vous invite à suivre [l'excellente documentation](https://gohugo.io/getting-started/quick-start/) de l'éditeur pour en créer un.

![hugo quickstart blog example](/2020/05/Hugo-Infomaniak-GithubActions/hugo-quickstart-blog.png) "Logo Title Text 1")