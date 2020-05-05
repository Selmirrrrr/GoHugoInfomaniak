---
layout: post
title: Personnaliser le "reverse engineering" d'Entity Framework Core
date: 2019-10-10T22:58:26+08:00
lastmod: 2019-10-19T22:58:26+08:00
tags: [EFCore, scaffold]
comments: true
---
Le ["reverse engineering" d'Entity Framework (Core)](https://docs.microsoft.com/en-us/ef/core/managing-schemas/scaffolding) génère un ensemble de classes et un DbContext représentant votre modèle de base de données permettant d'accéder à cette dernière. Dans cet article nous allons voir comment on peut personnaliser cette génération automatique et la faire correspondre à nos besoins.

## Contexte
Il y a quelque temps, sur [Stack Overflow](https://stackoverflow.com/questions/57261804/net-core-create-generic-repository-interface-id-mapping-for-all-tables-auto-cod/57436865#57436865), j'ai répondu à une question de quelqu'un souhaitant ajouter une interface `IEntity` à toutes les entités générées par l'outil.

Je vais donc détailler ma réponse dans cet article et expliquer pas à pas comment reproduire l'opération et l'adapter à vos besoins.

Cet article part volontairement du principe que vous avez une expérience certaine avec la génération de `DbContext`s avec Entity Framework (Core).

## Préparation
La première étape consiste à récupérer les sources de [Entity Framework Core](https://github.com/aspnet/EntityFrameworkCore) depuis GitHub. 

> Tout ceci est rendu possible parce que depuis .NET Core Microsoft a décidé d'open-sourcer tout le code du framework, sans cette décision nous n'aurions jamais pu récupérer les sources de l'outil et l'adapter à nos besoins. En grand fan de l'open-source que je suis, je ne peux que vous encourager vivement à participer et à vous plonger dans ce mouvement.

```
$ git clone https://github.com/aspnet/EntityFrameworkCore
```

Une fois le code récupéré, et avant toute modification, assurez-vous que le projet builde sur votre machine en exécutant le script de build à racine : 

```
$ build.cmd
```

Normalement, ça devrait fonctionner sans soucis, si cela devait ne pas être le cas, je vous invite à jeter un œil à la [doc du projet](https://github.com/aspnet/EntityFrameworkCore/wiki/getting-and-building-the-code).

## Découverte
Pour commencer, à la racine du repo ouvrez la solution `EFCore.Tools.slnf` (oui oui c'est bien [*.slnf*](https://docs.microsoft.com/en-us/visualstudio/ide/filtered-solutions?view=vs-2019)) et découvrez cette belle :heart_eyes: arborescence et organisation de solution, c'est magnifique, prenez-en de la graine ! Bref je m'égare...

Ensuite dans le projet `EFCore.Design` sous-dossier `Scaffolding\Internal` il y a une classe `CSharpEntityTypeGenerator` qui va particulièrement nous être utile aujourd'hui.

En effet, c'est cette classe qui va se charger d'écrire toutes les lignes de code qui vont composer nos entités, donc c'est ici qu'il va falloir implémenter les changements que l'on souhaite faire.

Si vous prenez un peu temps, vous pourrez constater qu'il y a d'autres classes très intéressantes dans ce dossier, comme par exemple `CSharpDbContextGenerator` qui pourrait nous permettre d'influer sur la génération de la classe `DbContext` de notre modèle.

## Implémentation
Dans le cadre de notre _"besoin"_ énoncé en introduction, nous voulons ajouter à nos classes générées une interface par défaut `IEntity` qui pourrait, par exemple, nous permettre de garantir que chacune de nos entités implémente un champ `Id`.

```csharp
public interface IEntity
{
    int Id { get; set; }
}
```

Pour ce faire, dans la classe `CSharpEntityTypeGenerator` dans la méthode `GenerateClass([NotNull] IEntityType entityType)` il y la ligne suivante : 

```csharp
_sb.AppendLine($"public partial class {entityType.Name}");
```

Comme on le voit c'est cette ligne qui va générer l'entête de notre classe :

```csharp
public partial class Customer
```

Pour ajouter notre interface il nous suffit donc de compléter la ligne avec `: IEntity` : 

```csharp
_sb.AppendLine($"public partial class {entityType.Name} : IEntity");
```

Voilà, c'est aussi simple que ça (bon vous me direz que le _"besoin"_ n'est pas foufou non plus :ghost:).

## Utilisation
Nous devons maintenant compiler et utiliser cette "version" custom d'Entity Framework Core dans notre projet perso afin de générer les modèles avec cette modification.

Commençons par builder le projet comme vu précédemment : 

```
$ build.cmd
```

Dans votre projet perso, ajouter une référence à la DLL `Microsoft.EntityFrameworkCore.Design` nouvellement buildée dans le dossier `artifacts\bin\EFCore.Design\Debug\netstandard2.1` du repo EFCore et exécuter la commande permettant de générer nos modèles :

```
$ dotnet ef dbcontext scaffold "DataSource=Northwind.sqlite" -o Models Microsoft.EntityFrameworkCore.Sqlite -f
```
_N'oubliez pas de remplacer `Microsoft.EntityFrameworkCore.Sqlite` par votre provider, `.SqlServer` pour MS SQL Server par exemple._

> `-f` permet d'overrider les modèles précédemment crées.    
> `-o` permet de placer les modèles dans le dossier `Models`.

Si vous n'êtes pas adeptes de la ligne de commande le résultat est le même en passant par la génération via Visual Studio comme vous en avez l'habitude.

## Résultat
Tous nos modèles générés implémentent maintenant notre interface comme on l'avait souhaité au départ : 

```csharp
public partial class Customer : IEntity
```

## Conclusion
Nous voilà arrivés au bout de ce petit exemple qui nous a permis de mettre notre petit grain de sel dans la tuyauterie de Entity Framework et ceci grâce au fait que le projet est open-source.

Bien sûr, cet exemple est minimaliste et très vite réalisé mais vous pouvez aller beaucoup plus loin et personnaliser à votre guise la génération de code du _scaffolding_ d'EF Core.

Par exemple, dernièrement, j'ai eu à générer un `DBContext` d'une base de données de plus de 200 tables et 3500 champs :fearful:. Par défaut EF Core mets toute la configuration (le mapping des champs aux propriétés) dans la classe `DbContext`.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    //...
    modelBuilder.Entity<Category>(entity =>
    {
        entity.Property(nameof(Category.Id)).ValueGeneratedNever();
        entity.Property(nameof(Category.CategoryName)).HasColumnType("VARCHAR(20)");
        entity.Property(nameof(Category.Description)).HasColumnType("VARCHAR(40)");
    });
    //...
}
```

Dans mon cas, cela produisait une classe de plus de 10'000 lignes :poop: qui mettait à mal mon instance de Visual Studio. C'est pourquoi j'ai décidé de créer une classe de configuration pour chaque entité : 

```csharp
public class CategoryConfiguration : IEntityTypeConfiguration<Category>
{
    public void Configure(EntityTypeBuilder<Category> entity)
    {
        entity.Property(nameof(Category.Id)).ValueGeneratedNever();
        entity.Property(nameof(Category.CategoryName)).HasColumnType("VARCHAR(20)");
        entity.Property(nameof(Category.Description)).HasColumnType("VARCHAR(40)");
    }
}
```

Et de simplement l'enregistrer dans le `DbContext`

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    //...
    modelBuilder.ApplyConfiguration(new CategoryConfiguration());
    //...
}
```

Du coup cela nous fait plus qu'une ligne par entité au lieu d'une (ou même souvent plusieurs) ligne(s) par champ dans le `DbContext`.

Bref, tout ça pour dire qu'avec un peu d'imagination et d'huile de coude, vous pouvez adapter le générateur de code à tous vos besoins (même les plus fous).

Happy hacking ! :call_me_hand:
