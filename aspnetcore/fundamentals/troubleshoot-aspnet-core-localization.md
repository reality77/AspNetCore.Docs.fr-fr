---
title: Résoudre les problèmes liés à la localisation ASP.NET Core
author: hishamco
description: Découvrez comment diagnostiquer les problèmes liés à la localisation dans les applications ASP.NET Core.
ms.author: riande
ms.date: 01/24/2019
uid: fundamentals/troubleshoot-aspnet-core-localization
ms.openlocfilehash: c76732c1a0389818f8f9efae8fe384ca0f9ca308
ms.sourcegitcommit: dd9c73db7853d87b566eef136d2162f648a43b85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/06/2019
ms.locfileid: "65087388"
---
# <a name="troubleshoot-aspnet-core-localization"></a>Résoudre les problèmes liés à la localisation ASP.NET Core

Par [Hisham Bin Ateya](https://github.com/hishamco)

Cet article fournit des instructions sur la façon de diagnostiquer les problèmes de localisation des applications ASP.NET Core.

## <a name="localization-configuration-issues"></a>Problèmes de configuration de la localisation

**Ordre de l’Intergiciel (middleware) de localisation**  
Il est possible que la localisation de l’application ne s’effectue pas car le middleware de localisation ne respecte pas l’ordre prévu.

Pour résoudre ce problème, vérifiez que le middleware de localisation est inscrit avant le middleware MVC. Sinon, le middleware de localisation n’est pas appliqué.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddLocalization(options => options.ResourcesPath = "Resources");

    services.AddMvc();
}
```

**Le chemin des ressources de localisation est introuvable**

**Les cultures prises en charge dans RequestCultureProvider ne correspondent pas à celles inscrites**  

## <a name="resource-file-naming-issues"></a>Problèmes liés au nommage des fichiers de ressources

ASP.NET Core a des règles prédéfinies et des instructions concernant le nommage des fichiers de ressources de localisation. Elles sont décrites en détail [ici](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming).

## <a name="missing-resources"></a>Ressources manquantes

Les causes courantes de ressources introuvables sont notamment les suivantes :

- Les noms des ressources sont mal orthographiés dans le fichier `resx` ou la requête à l’outil de localisation.
- La ressource est manquante dans le `resx` pour certaines langues, mais elle existe dans d’autres.
- Si le problème persiste, vérifiez les messages du journal de localisation (qui se trouvent au niveau du journal `Debug`) pour obtenir plus de détails sur les ressources manquantes.

_**Conseil :** Quand vous utilisez `CookieRequestCultureProvider`, vérifiez que des guillemets simples ne sont pas utilisés avec les cultures à l’intérieur de la valeur des cookies de localisation. Par exemple, `c='en-UK'|uic='en-US'` est une valeur de cookie non valide, tandis que `c=en-UK|uic=en-US` est valide._

## <a name="resources--class-libraries-issues"></a>Problèmes liés aux ressources et aux bibliothèques de classes

ASP.NET Core fournit par défaut un moyen de permettre aux bibliothèques de classes de rechercher leurs fichiers de ressources par le biais de [ResourceLocationAttribute](/dotnet/api/microsoft.extensions.localization.resourcelocationattribute?view=aspnetcore-2.1).

Les problèmes courants liés aux bibliothèques de classes sont les suivants :
- Le `ResourceLocationAttribute` manquant dans une classe de bibliothèque empêche `ResourceManagerStringLocalizerFactory` de découvrir les ressources.
- Nommage des fichiers de ressources. Pour plus d’informations, consultez la section [Problèmes liés au nommage des fichiers de ressources](#resource-file-naming-issues).
- Changement de l’espace de noms racine de la bibliothèque de classes. Pour plus d’informations, consultez la section [Problèmes liés à l’espace de noms racine](#root-namespace-issues).

## <a name="customrequestcultureprovider-doesnt-work-as-expected"></a>CustomRequestCultureProvider ne fonctionne pas comme prévu

La classe `RequestLocalizationOptions` possède trois fournisseurs par défaut :

1. `QueryStringRequestCultureProvider`
2. `CookieRequestCultureProvider`
3. `AcceptLanguageHeaderRequestCultureProvider`

[CustomRequestCultureProvider](/dotnet/api/microsoft.aspnetcore.localization.customrequestcultureprovider?view=aspnetcore-2.1) vous permet de personnaliser la façon dont la culture de localisation est fournie dans votre application. `CustomRequestCultureProvider` est utilisé quand les fournisseurs par défaut ne répondent pas à vos besoins.

- Une raison courante pour laquelle un fournisseur personnalisé ne fonctionne pas correctement est qu’il n’est pas le premier fournisseur dans la liste `RequestCultureProviders`. Pour résoudre ce problème :

- Insérez le fournisseur personnalisé à la position 0 dans la liste `RequestCultureProviders`, comme suit :

```csharp
options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
```

- Utilisez la méthode d’extension `AddInitialRequestCultureProvider` pour définir le fournisseur personnalisé comme fournisseur initial.

## <a name="root-namespace-issues"></a>Problèmes liés à l’espace de noms racine

Si l’espace de noms racine d’un assembly est différent du nom de l’assembly, la localisation ne fonctionne pas par défaut. Pour éviter ce problème, utilisez [RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1), qui est décrit en détail [ici](xref:fundamentals/localization?view=aspnetcore-2.2#resource-file-naming).

## <a name="resources--build-action"></a>Ressources et action de génération

Si vous utilisez des fichiers de ressources pour la localisation, il est important qu’ils disposent d’une action de génération appropriée. Ils doivent être une **ressource incorporée**, sans quoi `ResourceStringLocalizer` n’est pas en mesure de trouver ces ressources.
