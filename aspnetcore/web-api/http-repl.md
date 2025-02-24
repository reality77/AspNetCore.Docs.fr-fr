---
title: Tester des API web avec la boucle REPL HTTP
author: scottaddie
description: Découvrez comment utiliser l’outil global REPL HTTP de .NET Core pour parcourir et tester une API web ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 08/29/2019
uid: web-api/http-repl
ms.openlocfilehash: 086ac141a04ab4a560f2c26fb049ef8a5493dc97
ms.sourcegitcommit: d34b2627a69bc8940b76a949de830335db9701d3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2019
ms.locfileid: "71187238"
---
# <a name="test-web-apis-with-the-http-repl"></a>Tester des API web avec la boucle REPL HTTP

Par [Scott Addie](https://twitter.com/Scott_Addie)

La boucle REPL (Read-Eval-Print Loop) HTTP est :

* Un outil en ligne de commande léger et multiplateforme qui est pris en charge partout où .NET Core est pris en charge.
* Utilisée pour effectuer des requêtes HTTP pour tester des API web ASP.NET Core (et des API web ASP.NET Core non-ASP.NET) et voir leurs résultats.
* Capable de tester les API web hébergées dans n’importe quel environnement, notamment localhost et Azure App Service.

Les [verbes HTTP](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods) suivants sont pris en charge :

* [DELETE](#test-http-delete-requests)
* [TÉLÉCHARGER](#test-http-get-requests)
* [HEAD](#test-http-head-requests)
* [OPTIONS](#test-http-options-requests)
* [CORRECTIF](#test-http-patch-requests)
* [POST](#test-http-post-requests)
* [PUT](#test-http-put-requests)

Pour continuer, [consultez ou téléchargez l’exemple d’API web ASP.NET Core](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples) ([comment télécharger](xref:index#how-to-download-a-sample)).

## <a name="prerequisites"></a>Prérequis

* [!INCLUDE [2.1-SDK](~/includes/2.1-SDK.md)]

## <a name="installation"></a>Installation

Pour installer la boucle REPL HTTP, exécutez la commande suivante :

```dotnetcli
dotnet tool install -g Microsoft.dotnet-httprepl
```

Un [outil global .NET Core](/dotnet/core/tools/global-tools#install-a-global-tool) est installé à partir du package NuGet [Microsoft.dotnet-httprepl](https://www.nuget.org/packages/Microsoft.dotnet-httprepl).

## <a name="usage"></a>Utilisation

Une fois l’installation de l’outil réussie, exécutez la commande suivante pour démarrer la boucle REPL HTTP :

```console
httprepl
```

Pour voir les commandes de la boucle REPL HTTP disponibles, exécutez une des commandes suivantes :

```console
httprepl -h
```

```console
httprepl --help
```

La sortie suivante s’affiche :

```console
Usage:
  httprepl [<BASE_ADDRESS>] [options]

Arguments:
  <BASE_ADDRESS> - The initial base address for the REPL.

Options:
  -h|--help - Show help information.

Once the REPL starts, these commands are valid:

Setup Commands:
Use these commands to configure the tool for your API server

connect        Configures the directory structure and base address of the api server
set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`

HTTP Commands:
Use these commands to execute requests against your application.

GET            get - Issues a GET request
POST           post - Issues a POST request
PUT            put - Issues a PUT request
DELETE         delete - Issues a DELETE request
PATCH          patch - Issues a PATCH request
HEAD           head - Issues a HEAD request
OPTIONS        options - Issues a OPTIONS request

Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

set base       Set the base URI. e.g. `set base http://locahost:5000`
ls             Show all endpoints for the current path
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands
exit           Exit the shell

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\\Program Files\\Microsoft VS Code\\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line
ui             Displays the Swagger UI page, if available, in the default browser

Use `help <COMMAND>` for more detail on an individual command. e.g. `help get`.
For detailed tool info, see https://aka.ms/http-repl-doc.
```

La boucle REPL HTTP offre la complétion des commandes. Un appui sur la touche <kbd>Tab</kbd> itère au sein de la liste des commandes qui complètent les caractères ou le point de terminaison d’API que vous avez tapés. Les sections suivantes décrivent les commandes CLI disponibles.

## <a name="connect-to-the-web-api"></a>Se connecter à l’API web

Connectez-vous à une API web en exécutant la commande suivante :

```console
httprepl <ROOT URI>
```

`<ROOT URI>` est l’URI de base pour l’API web. Par exemple :

```console
httprepl https://localhost:5001
```

Vous pouvez également exécuter la commande suivante à tout moment pendant l’exécution de la boucle REPL HTTP :

```console
connect <ROOT URI>
```

Par exemple :

```console
(Disconnected)~ connect https://localhost:5001
```

## <a name="manually-point-to-the-swagger-document-for-the-web-api"></a>Pointer manuellement sur le document Swagger pour l’API web

La commande connect ci-dessus tente de trouver automatiquement le document Swagger. Si, pour une raison quelconque, elle n’y parvient pas, vous pouvez spécifier l’URI du document Swagger pour l’API web à l’aide de l’option `--swagger` :

```console
connect <ROOT URI> --swagger <SWAGGER URI>
```

Par exemple :

```console
(Disconnected)~ connect https://localhost:5001 --swagger /swagger/v1/swagger.json
```

## <a name="navigate-the-web-api"></a>Accéder à l’API web

### <a name="view-available-endpoints"></a>Voir les points de terminaison disponibles

Pour lister les différents points de terminaison (contrôleurs) sur le chemin actuel de l’adresse de l’API web, exécutez la commande `ls` ou `dir` :

```console
https://localhot:5001/~ ls
```

Le format de sortie suivant s’affiche :

```console
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

La sortie précédente indique que deux contrôleurs sont disponibles : `Fruits` et `People`. Les deux contrôleurs prennent en charge les opérations HTTP GET et POST sans paramètres.

La navigation dans un contrôleur spécifique révèle plus de détails. Par exemple, la sortie de la commande suivante indique que le contrôleur `Fruits` prend également en charge les opérations HTTP GET, PUT et DELETE. Chacune de ces opérations attend un paramètre `id` dans la route :

```console
https://localhost:5001/fruits~ ls
.      [get|post]
..     []
{id}   [get|put|delete]

https://localhost:5001/fruits~
```

Vous pouvez également exécuter la commande `ui` pour ouvrir la page de l’interface utilisateur Swagger de l’API web dans un navigateur. Par exemple :

```console
https://localhost:5001/~ ui
```

### <a name="navigate-to-an-endpoint"></a>Accéder à un point de terminaison

Pour accéder à un autre point de terminaison sur l’API web, exécutez la commande `cd` :

```console
https://localhost:5001/~ cd people
```

Le chemin qui suit la commande `cd` ne respecte pas la casse. Le format de sortie suivant s’affiche :

```console
/people    [get|post]

https://localhost:5001/people~
```

## <a name="customize-the-http-repl"></a>Personnaliser la boucle REPL HTTP

Les [couleurs](#set-color-preferences) de la boucle REPL HTTP peuvent être personnalisées. Vous pouvez aussi définir un [éditeur de texte par défaut](#set-the-default-text-editor). Les préférences de la boucle REPL HTTP sont conservées dans la session active et dans les sessions ultérieures. Une fois modifiées, les préférences sont stockées dans le fichier suivant :

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

*%HOME%/.httpreplprefs*

# <a name="macostabmacos"></a>[macOS](#tab/macos)

*%HOME%/.httpreplprefs*

# <a name="windowstabwindows"></a>[Fenêtres](#tab/windows)

*%USERPROFILE%\\.httpreplprefs*

---

Le fichier *.httpreplprefs* est chargé au démarrage et ses modifications ne sont pas surveillées pendant l’exécution. Les modifications manuelles apportées au fichier prennent effet seulement après un redémarrage de l’outil.

### <a name="view-the-settings"></a>Voir les paramètres

Pour voir les paramètres disponibles, exécutez la commande `pref get`. Par exemple :

```console
https://localhost:5001/~ pref get
```

La commande précédente affiche les paires clé-valeur disponibles :

```console
colors.json=Green
colors.json.arrayBrace=BoldCyan
colors.json.comma=BoldYellow
colors.json.name=BoldMagenta
colors.json.nameSeparator=BoldWhite
colors.json.objectBrace=Cyan
colors.protocol=BoldGreen
colors.status=BoldYellow
```

### <a name="set-color-preferences"></a>Définir les préférences de couleur

La colorisation des réponses est actuellement prise en charge seulement pour JSON. Pour personnaliser les couleurs par défaut de l’outil REPL HTTP, localisez la clé correspondant à la couleur à changer. Pour des instructions sur la façon de trouver les clés, consultez la section [Voir les paramètres](#view-the-settings). Par exemple, remplacez la valeur de la clé `colors.json` de `Green` par `White` :

```console
https://localhost:5001/people~ pref set colors.json White
```

Seules les [couleurs autorisées](https://github.com/aspnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs) peuvent être utilisées. Les requêtes HTTP suivantes affichent la sortie avec les nouvelles couleurs.

Quand des clés d’une couleur spécifique ne sont pas définies, des clés plus génériques sont prises en compte. Pour illustrer ce comportement de repli, considérez l’exemple suivant :

* Si `colors.json.name` n’a pas de valeur, `colors.json.string` est utilisée.
* Si `colors.json.string` n’a pas de valeur, `colors.json.literal` est utilisée.
* Si `colors.json.literal` n’a pas de valeur, `colors.json` est utilisée. 
* Si `colors.json` n’a pas de valeur, la couleur de texte par défaut de l’interpréteur de commandes (`AllowedColors.None`) est utilisée.

### <a name="set-indentation-size"></a>Définir la taille de la mise en retrait

La personnalisation de la taille de la mise en retrait de la réponse est actuellement prise en charge pour JSON uniquement. La taille par défaut est de deux espaces. Par exemple :

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Pour changer la taille par défaut, définissez la clé `formatting.json.indentSize`. Par exemple, pour toujours utiliser quatre espaces :

```console
pref set formatting.json.indentSize 4
```

Les réponses suivantes adoptent la valeur correspondant à quatre espaces :

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-the-default-text-editor"></a>Définir l’éditeur de texte par défaut

Par défaut, la boucle REPL HTTP n’a pas d’éditeur de texte configuré pour l’utilisation. Pour tester les méthodes d’API web nécessitant un corps de requête HTTP, vous devez définir un éditeur de texte par défaut. L’outil REPL HTTP lance l’éditeur de texte configuré dans le seul but de permettre la composition du corps de la requête. Exécutez la commande suivante pour définir votre éditeur de texte préféré comme éditeur par défaut :

```console
pref set editor.command.default "<EXECUTABLE>"
```

Dans la commande précédente, `<EXECUTABLE>` est le chemin complet du fichier exécutable de l’éditeur de texte. Par exemple, exécutez la commande suivante pour définir Visual Studio Code comme éditeur de texte par défaut :

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```console
pref set editor.command.default "/usr/bin/code"
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

```console
pref set editor.command.default "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code"
```

# <a name="windowstabwindows"></a>[Fenêtres](#tab/windows)

```console
pref set editor.command.default "C:\Program Files\Microsoft VS Code\Code.exe"
```

---

Pour lancer l’éditeur de texte par défaut avec des arguments CLI spécifiques, définissez la clé `editor.command.default.arguments`. Par exemple, supposons que Visual Studio Code est l’éditeur de texte par défaut et que vous voulez que la boucle REPL HTTP ouvre toujours Visual Studio Code dans une nouvelle session avec les extensions désactivées. Exécutez la commande suivante :

```console
pref set editor.command.default.arguments "--disable-extensions --new-window"
```

### <a name="set-the-swagger-search-paths"></a>Définir les chemins de recherche Swagger

Par défaut, HTTP REPL possède un ensemble de chemins relatifs qu’il utilise pour rechercher le document Swagger lors de l'exécution de la commande `connect` sans l’option `--swagger`. Ces chemins relatifs sont combinés avec les chemins racine et de base spécifiés dans la commande `connect`. Les chemins relatifs par défaut sont les suivants :

- *swagger.json*
- *swagger/v1/swagger.json*
- */swagger.json*
- */swagger/v1/swagger.json*

Pour utiliser un autre ensemble de chemins de recherche dans votre environnement, définissez la préférence `swagger.searchPaths`. La valeur doit être une liste de chemins relatifs délimités par des barres verticales. Par exemple :

```console
pref set swagger.searchPaths "swagger/v2/swagger.json|swagger/v3/swagger.json"
```

## <a name="test-http-get-requests"></a>Tester des requêtes HTTP GET

### <a name="synopsis"></a>Résumé

```console
get <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

Les options suivantes sont disponibles pour la commande `get` :

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Exemple

Pour émettre une requête HTTP GET :

1. Exécutez la commande `get` sur un point de terminaison qui la prend en charge :

    ```console
    https://localhost:5001/people~ get
    ```

    La commande précédente affiche le format de sortie suivant :

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 03:38:45 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "name": "Scott Hunter"
      },
      {
        "id": 2,
        "name": "Scott Hanselman"
      },
      {
        "id": 3,
        "name": "Scott Guthrie"
      }
    ]


    https://localhost:5001/people~
    ```

1. Récupérez un enregistrement spécifique en passant un paramètre à la commande `get` :

    ```console
    https://localhost:5001/people~ get 2
    ```

    La commande précédente affiche le format de sortie suivant :

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 06:17:57 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 2,
        "name": "Scott Hanselman"
      }
    ]


    https://localhost:5001/people~
    ```

## <a name="test-http-post-requests"></a>Tester des requêtes HTTP POST

### <a name="synopsis"></a>Résumé

```console
post <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Exemple

Pour émettre une requête HTTP POST :

1. Exécutez la commande `post` sur un point de terminaison qui la prend en charge :

    ```console
    https://localhost:5001/people~ post -h Content-Type=application/json
    ```

    Dans la commande précédente, l’en-tête `Content-Type` de la requête HTTP est défini pour indiquer un type de média de corps de requête JSON. L’éditeur de texte par défaut ouvre un fichier *.tmp* avec un modèle JSON représentant le corps de la requête HTTP. Par exemple :

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Pour définir l’éditeur de texte par défaut, consultez la section [Définir l’éditeur de texte par défaut](#set-the-default-text-editor).

1. Modifiez le modèle JSON pour répondre aux exigences de validation du modèle :

    ```json
    {
      "id": 0,
      "name": "Scott Addie"
    }
    ```

1. Enregistrez le fichier *.tmp* et fermez l’éditeur de texte. La sortie suivante s’affiche dans l’interpréteur de commandes :

    ```console
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Thu, 27 Jun 2019 21:24:18 GMT
    Location: https://localhost:5001/people/4
    Server: Kestrel
    Transfer-Encoding: chunked

    {
      "id": 4,
      "name": "Scott Addie"
    }


    https://localhost:5001/people~
    ```

## <a name="test-http-put-requests"></a>Tester des requêtes HTTP PUT

### <a name="synopsis"></a>Résumé

```console
put <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Exemple

Pour émettre une requête HTTP PUT :

1. *Facultatif* : Exécutez la commande `get` pour afficher les données avant de les modifier :

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `put` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ put 2 -h Content-Type=application/json
    ```

    Dans la commande précédente, l’en-tête `Content-Type` de la requête HTTP est défini pour indiquer un type de média de corps de requête JSON. L’éditeur de texte par défaut ouvre un fichier *.tmp* avec un modèle JSON représentant le corps de la requête HTTP. Par exemple :

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Pour définir l’éditeur de texte par défaut, consultez la section [Définir l’éditeur de texte par défaut](#set-the-default-text-editor).

1. Modifiez le modèle JSON pour répondre aux exigences de validation du modèle :

    ```json
    {
      "id": 2,
      "name": "Cherry"
    }
    ```

1. Enregistrez le fichier *.tmp* et fermez l’éditeur de texte. La sortie suivante s’affiche dans l’interpréteur de commandes :

    ```console
    [main 2019-06-28T17:27:01.805Z] update#setState idle
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:28:21 GMT
    Server: Kestrel
    ```

1. *Facultatif* : Émettez une commande `get` pour voir les modifications. Par exemple, si vous avez tapé « Cherry » dans l’éditeur de texte, une commande `get` retourne ce qui suit :

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:08:20 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Cherry"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-delete-requests"></a>Tester des requêtes HTTP DELETE

### <a name="synopsis"></a>Résumé

```console
delete <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Exemple

Pour émettre une requête HTTP DELETE :

1. *Facultatif* : Exécutez la commande `get` pour voir les données avant de les modifier :

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]

1. Run the `delete` command on an endpoint that supports it:

    ```console
    https://localhost:5001/fruits~ delete 2
    ```

    La commande précédente affiche le format de sortie suivant :

    ```console
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:36:42 GMT
    Server: Kestrel
    ```

1. *Facultatif* : Émettez une commande `get` pour voir les modifications. Dans cet exemple, une commande `get` retourne ce qui suit :

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:16:30 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-patch-requests"></a>Tester des requêtes HTTP PATCH

### <a name="synopsis"></a>Résumé

```console
patch <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

## <a name="test-http-head-requests"></a>Tester des requêtes HTTP HEAD

### <a name="synopsis"></a>Résumé

```console
head <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="test-http-options-requests"></a>Tester des requêtes HTTP OPTIONS

### <a name="synopsis"></a>Résumé

```console
options <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Paramètre de route, le cas échéant, attendu par la méthode d’action du contrôleur associé.

### <a name="options"></a>Options

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="set-http-request-headers"></a>Définir des en-têtes de requête HTTP

Pour définir un en-tête de requête HTTP, utilisez une des approches suivantes :

1. Définir inline avec la requête HTTP. Par exemple :

  ```console
  https://localhost:5001/people~ post -h Content-Type=application/json
  ```

  Avec l’approche précédente, chaque en-tête de requête HTTP distinct nécessite sa propre option `-h`.

1. Définir avant l’envoi de la requête HTTP. Par exemple :

  ```console
  https://localhost:5001/people~ set header Content-Type application/json
  ```

  Si l’en-tête est défini avant l’envoi d’une requête, l’en-tête reste défini pour la durée de la session de l’interpréteur de commandes. Pour effacer l’en-tête, spécifiez une valeur vide. Par exemple :

  ```console
  https://localhost:5001/people~ set header Content-Type
  ```

## <a name="toggle-http-request-display"></a>Activer/désactiver l’affichage des requêtes HTTP

Par défaut, l’affichage de la requête HTTP envoyée est supprimé. Il est possible de changer le paramètre correspondant pour la durée de la session de l’interpréteur de commandes.

### <a name="enable-request-display"></a>Activer l’affichage des requêtes

Affichez la requête HTTP envoyée en exécutant la commande `echo on`. Par exemple :

```console
https://localhost:5001/people~ echo on
Request echoing is on
```

Les requêtes HTTP suivantes dans la session active affichent les en-têtes de requête. Par exemple :

```console
https://localhost:5001/people~ post

[main 2019-06-28T18:50:11.930Z] update#setState idle
Request to https://localhost:5001...

POST /people HTTP/1.1
Content-Length: 41
Content-Type: application/json
User-Agent: HTTP-REPL

{
  "id": 0,
  "name": "Scott Addie"
}

Response from https://localhost:5001...

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Jun 2019 18:50:21 GMT
Location: https://localhost:5001/people/4
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Scott Addie"
}


https://localhost:5001/people~
```

### <a name="disable-request-display"></a>Désactiver l’affichage des requêtes

Supprimez l’affichage de la requête HTTP envoyée en exécutant la commande `echo off`. Par exemple :

```console
https://localhost:5001/people~ echo off
Request echoing is off
```

## <a name="run-a-script"></a>Exécuter un script

Si vous exécutez fréquemment le même jeu de commandes REPL HTTP, envisagez de les stocker dans un fichier texte. Les commandes placées dans le fichier sont de la même forme que celles exécutées manuellement sur la ligne de commande. Les commandes peuvent être exécutées de façon groupée avec la commande `run`. Par exemple :

1. Créez un fichier texte contenant un ensemble de commandes délimitées par des sauts de ligne. Pour illustrer ceci, considérez un fichier *people-script.txt* contenant les commandes suivantes :

    ```text
    set base https://localhost:5001
    ls
    cd People
    ls
    get 1
    ```

1. Exécutez la commande `run`, en passant le chemin du fichier texte. Par exemple :

    ```console
    https://localhost:5001/~ run C:\http-repl-scripts\people-script.txt
    ```

    La sortie suivante apparaît :

    ```console
    https://localhost:5001/~ set base https://localhost:5001
    Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json
    
    https://localhost:5001/~ ls
    .        []
    Fruits   [get|post]
    People   [get|post]
    
    https://localhost:5001/~ cd People
    /People    [get|post]
    
    https://localhost:5001/People~ ls
    .      [get|post]
    ..     []
    {id}   [get|put|delete]
    
    https://localhost:5001/People~ get 1
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 12 Jul 2019 19:20:10 GMT
    Server: Kestrel
    Transfer-Encoding: chunked
    
    {
      "id": 1,
      "name": "Scott Hunter"
    }
    
    
    https://localhost:5001/People~
    ```

## <a name="clear-the-output"></a>Effacer la sortie

Pour supprimer toutes les sorties écrites dans l’interpréteur de commandes par l’outil REPL HTTP, exécutez la commande `clear` ou `cls`. À titre d’illustration, imaginez que l’interpréteur de commandes contient la sortie suivante :

```console
httprepl https://localhost:5001
(Disconnected)~ set base "https://localhost:5001"
Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json

https://localhost:5001/~ ls
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Exécutez la commande suivante pour effacer la sortie :

```console
https://localhost:5001/~ clear
```

Après l’exécution de la commande précédente, l’interpréteur de commandes contient seulement la sortie suivante :

```console
https://localhost:5001/~
```

## <a name="additional-resources"></a>Ressources supplémentaires

* [Requêtes de l’API REST](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods)
* [Dépôt GitHub REPL HTTP](https://github.com/aspnet/HttpRepl)
