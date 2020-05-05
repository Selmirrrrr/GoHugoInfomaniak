---
layout: post
title: Migrer de TFVC (TFS) à Git
subtitle: Microsoft ne nous dit pas tout...
tags: [git, tfs, tfvc]
date: 2019-10-07T19:10:26+08:00
lastmod: 2019-10-07T19:10:26+08:00
comments: true
featuredImage: "/images/gitbanner.png"

---
[TFS (Team Foundation Server)](https://en.wikipedia.org/wiki/Team_Foundation_Server) ne désigne pas un source control mais l'ensemble d'outils que Microsoft met à disposition pour la gestion du code source, la gestion de projet, l'automatisation de build, le testing automatisé, le déploiement, ~le café~, etc...

[TFVC (Team Foundation Version Control)](https://docs.microsoft.com/en-us/azure/devops/repos/tfvc/overview?view=azure-devops) désigne quant à lui le système de gestion de sources. TFS, peut fonctionner avec TFVC ou Git pour gérer les sources, TFS n'est pas forcément = Git.

Ces derniers jours plusieurs personnes m'ont parlé de leur source control sous TFVC et de leur impossibilité de migrer leurs sources à Git.

Étant personnellement un très grand fan de Git et ayant été en charge d'une migration de sources de TFVC et SVN à Git et ayant été à l'origine de cette décision de migration j'ai décidé de rédiger cette article afin d'aider le plus grand nombre à se débarrasser de cet outil dépassé et obsolète qu'est TFVC. Même Microsoft n'utilise pas TFVC mais Git, donc n'attendez pas, migrez aussi !

Cet article n'aborde volontairement pas la comparaison entre TFVC et Git, il y a bien assez de ressources disponible sur le net pour ça, mais en cas besoin je rédigerai un second article pour aborder ce point.

## La migration selon MS
La principale raison de pourquoi les gens ont du mal à faire le saut entre TFVC et Git est la "peur" instaurée par Microsoft, regardez plutôt [les préréquis indiqués(https://docs.microsoft.com/en-us/azure/devops/learn/git/migrate-from-tfvc-to-git#requirements)].

 - On ne peut migrer qu'une seule branche
 - On ne peut conserver que les 180 derniers jours d'historique
 - Le repository ne peut excéder 1GB au total

Bien contraignant tout ça... Donc, à en croire Microsoft, pour migrer de TFVC à Git il faut arrêter le dév de toute une équipe et être prêt à perdre tout l'historique plus antérieur aux 6 derniers mois.

Je vous rassure toute suite, vous n'aurez pas du tout à faire toutes ces concessions afin de migrer votre code source à Git.

## La migration selon Matt Burke
Matt Burke ne vous dit sûrement rien, mais c'est le créateur du projet [git-tfs](https://github.com/git-tfs/git-tfs) et c'est grâce à lui (entre-autres) et son travail pour la communauté que vous allez pouvoir migrer code base à Git tranquillement.

### 1. Installation
Rendez-vous sur [git-tfs.com](https://git-tfs.com/) et téléchargez la dernière version de l'outil.

Après l'installation, pour vérifier que l'outil est bien installé, dans l'invité de commande exécutez : 

```
git tfs help
```

### 2. Préparation
Il n'y rien à préparer, vous pouvez commencer la migration dès maintenant, oui, oui, même si vos collègues continuent à travailler sur le répo TFVC.

### 3. Le clone
La commande ci-dessous va parcourir tous les commits de votre TFVC et le rejouer sur un répo Git, du coup elle peut prendre pas mal de temps si vous avez un très gros répo.

```
git tfs clone https://tfs.comoany.com/tfs/Collection $/prjName/trunk . --branches=all
```

A titre de comparaison, cette commande a pris 4h pour une répo de ~6'000 commits dans mon cas, donc armez-vous de patience.

Une fois ce premier clone terminé, exécutez la commande suivante dans votre nouveau répo Git pour récupérer les branches "complexes" qui n'ont pas pu l'autre lors de la première commande.

```
git tfs branch –init --all
```

Maintenant c'est fait, si tout s'est bien déroulé vous avez en local tout votre historique des changements et toutes les branches de votre ancien répo TFVC.

> Héy mon coco, ma migration a duré 4h et comme t'as dit que l'on n'était pas obligé d'arrêter les équipes j'ai des nouveaux commits qui sont apparus sur TFVC mais que je n'ai pas dans mon répo Git.

{: .box-note}
**Note:** Pas de soucis, je ne vous ai pas menti, voici la solution !

### 4. Mise à jour
Pour chaque branche où il y a eu du changement exécuter les commandes suivantes : 

```
git checkout NOM_DE_LA_BRANCHE
```

```
git tfs pull
```

Ceci va récupérer les changements appliqués entre-temps sur votre repo, ça devrait donc aller assez vite.

### 5. Le push
Maintenant que notre répo Git en local est prêt, nous aller le "pusher" sur un hébergeur de répo centralisé type GitHub, GitLab, BitBucket, Azure DevOps(TFS), etc.

Je pars du principe que vous avec déjà fait le nécessaire de ce côté et que vous avec déjà créer un répo vide sur le système de votre choix.

Il nous reste maintenant plus qu'à pousser notre répo afin que le reste de l'équipe puisse commencer à travailler dessus : 

```
git remote add origin https://git.company.com/PROJECT/repo.git
```

```
git push origin -u -all
```

### 6. Conclusion
C'est fait, la migration "technique" il nous vous reste "plus qu'à" (d'expérience, ce n'est pas la partie la plus facile) faire passer vos collègues de TFVC à Git et à passer votre ancien TFVC en lecture seule pour éviter d'avoir des gens qui continuent à travailler sur cet outil.

## Mes conseils
 - Préparez la migration avec vos développeurs, même si Git est N fois mieux que TFVC il peut être difficile pour un développeur de changer ses outils et ses habitudes, préparez vos équipes à ce changement avec des formations, des workshops, etc.
 - Ne faites pas l'erreur qu'une majeure partie des gens font, ne vous lancez pas toute de suite dans un système de branching compliqué type GitFlow ou autres. Certes, Git le permet et est très puissant pour ça, mais vous venez de TFVC, un monde où beaucoup de gens ne travaillent qu'avec le "trunk", partez donc sur un système branche simple (type [Trunk based](https://devblogs.microsoft.com/devops/release-flow-how-we-do-branching-on-the-vsts-team/)) et faites le évoluer dans le temps si besoin.
 - Profitez de la migration pour "nettoyer" vos sources.