---
title: Déployer votre bot | Microsoft Docs
description: Déployer votre bot dans le cloud Azure
keywords: déployer bot, déployer bot azure, publier bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 08/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d44b1339b367c7243cfb6d2311a553becb6245ff
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037505"
---
# <a name="deploy-your-bot"></a>Déployer votre bot

[!INCLUDE [applies-to](./includes/applies-to.md)]

Dans cet article, nous vous montrons comment déployer un bot simple dans Azure. Nous vous expliquons comment préparer votre bot en vue de son déploiement, comment le déployer dans Azure et comment le tester dans Web Chat. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.

<!-- create your Azure Application, 2) prepare your source code for deployment, and 3) deploy your code to your Azure Application.  -->

## <a name="prerequisites"></a>Prérequis
- Un abonnement à [Microsoft Azure](https://azure.microsoft.com/free/).
- Un bot C#, JavaScript ou TypeScript que vous avez développé sur votre machine locale.
- La version la plus récente d’[Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).
- De bonnes connaissances sur [l’interface Azure CLI et les modèles ARM](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview).

## <a name="prepare-for-deployment"></a>Préparation du déploiement
Quand vous créez un bot à l’aide d’un [modèle Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) ou d’un [modèle Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0), le code source généré contient un dossier `deploymentTemplates` renfermant des modèles ARM. Le processus de déploiement documenté ici utilise le modèle ARM pour provisionner les ressources nécessaires pour le bot dans Azure à l’aide de l’interface Azure CLI. 

> [!NOTE]
> Avec la publication du SDK Bot Framework 4.3, nous avons _déprécié_ l’utilisation du fichier .bot en faveur d’un fichier appsettings.json ou .env pour la gestion des ressources. Pour plus d’informations sur la migration des paramètres du fichier .bot vers le fichier appsettings.json ou .env, consultez [Gérer les ressources du bot](v4sdk/bot-file-basics.md).

### <a name="1-login-to-azure"></a>1. Connexion à Azure

Vous avez déjà créé et testé un bot localement et vous souhaitez maintenant le déployer sur Azure. Ouvrez une invite de commandes pour vous connecter au portail Azure.

```cmd
az login
```
Une nouvelle fenêtre de navigateur s’ouvre ; connectez-vous.

> [!NOTE]
> Si vous déployez votre bot sur un cloud autre qu’Azure comme Gov (US), vous devez exécuter `az cloud set --name <name-of-cloud>` avant `az login`, où &lt;nom-de-cloud> est le nom d’un cloud inscrit, par exemple `AzureUSGovernment`. Si vous souhaitez revenir au cloud public, vous pouvez exécuter `az cloud set --name AzureCloud`. 

### <a name="2-set-the-subscription"></a>2. Définir l’abonnement

Définissez l’abonnement par défaut à utiliser.

```cmd
az account set --subscription "<azure-subscription>"
```

Si vous ne savez pas quel abonnement utiliser pour déployer le bot, vous pouvez consulter la liste des abonnements pour votre compte à l’aide de la commande `az account list`. Accédez au dossier bot.

### <a name="3-create-an-app-registration"></a>3. Créer une inscription d’application

Inscrire l’application signifie que vous pouvez utiliser Azure AD pour authentifier les utilisateurs et demander l’accès aux ressources utilisateur. Votre bot a besoin d’une application inscrite dans Azure qui permet au bot d’accéder à Bot Framework Service pour l’envoi et la réception de messages authentifiés. Pour créer et inscrire une application par le biais de l’interface Azure CLI, exécutez la commande suivante :

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Option   | Description |
|:---------|:------------|
| display-name | Nom d’affichage de l’application. |
| password | Mot de passe de l’application, également appelé « secret client ». Le mot de passe doit comporter au moins 16 caractères, contenir au moins un caractère alphabétique minuscule ou majuscule, et contenir au moins un caractère spécial.|
| available-to-other-tenants| L’application peut être utilisée à partir de n’importe quel locataire Azure AD. Ce paramètre doit être `true` pour permettre à votre bot de fonctionner avec les canaux Azure Bot Service.|

La commande ci-dessus génère du code JSON avec la clé `appId` et enregistre la valeur de cette clé pour le déploiement ARM, où elle sera utilisée pour le paramètre `appId`. Le mot de passe fourni sera utilisé pour le paramètre `appSecret`.

> [!NOTE] 
> Si vous souhaitez utiliser une inscription d’application existante, vous pouvez utiliser la commande :
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```

### <a name="4-deploy-via-arm-template"></a>4. Déployer le bot avec un modèle ARM
Vous pouvez déployer votre bot dans un groupe de ressources nouveau ou existant. Choisissez l’option qui vous convient le mieux. 
<!--
## [Deploy via ARM template (with **new**  Resource Group)](#tab/nerg)
-->
#### <a name="deploy-via-arm-template-with-new-resource-group"></a>**Déployer par le biais du modèle ARM (avec un **nouveau** groupe de ressources)**
<!-- ##### Create Azure resources -->
Vous allez créer un groupe de ressources dans Azure, puis utiliser le modèle ARM pour créer les ressources qui y sont spécifiées. Ici, nous spécifions le plan App Service, l’application web et une inscription Bot Channels Registration.

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| Option   | Description |
|:---------|:------------|
| Nom | Nom convivial pour le déploiement. |
| template-file | Chemin du modèle ARM. Vous pouvez utiliser le fichier `template-with-new-rg.json` fourni dans le dossier `deploymentTemplates` du projet. |
| location |Lieu. Valeurs provenant de : `az account list-locations`. Vous pouvez configurer le lieu par défaut en utilisant `az configure --defaults location=<location>`. |
| parameters | Spécifiez les valeurs des paramètres du déploiement. Valeur `appId` que vous avez obtenue en exécutant la commande `az ad app create`. `appSecret` est le mot de passe que vous avez fourni à l’étape précédente. Le paramètre `botId`, qui est utilisé comme ID de bot immuable, doit être globalement unique. Il sert aussi à configurer le nom d’affichage du bot, qui est mutable. `botSku` est le niveau tarifaire ; il peut s’agir de F0 (gratuit) ou S1 (Standard). `newAppServicePlanName` est le nom du plan App Service. `newWebAppName` est le nom de l’application web que vous créez. `groupName` est le nom du groupe de ressources Azure que vous créez. `groupLocation` est l’emplacement du groupe de ressources Azure. `newAppServicePlanLocation` est l’emplacement du plan App Service. |

#### <a name="deploy-via-arm-template-with-existing--resource-group"></a>**Déployer par le biais du modèle ARM (avec un groupe de ressources **existant**)**
<!--
## [Deploy via ARM template (with **existing**  Resource Group)](#tab/erg)
##### Create Azure resources
-->
Quand vous utilisez un groupe de ressources existant, vous pouvez utiliser un plan App Service existant ou en créer un. Les étapes de ces deux options sont indiquées ci-dessous. 

**Option 1 : Plan App Service existant** 

Ici, nous utilisons un plan App Service existant, mais nous créons une application web et une inscription Bot Channels Registration. 

> [!NOTE]
> Le paramètre botId, qui est utilisé comme ID de bot immuable, doit être globalement unique. Il sert aussi à configurer le nom d’affichage du bot, qui est mutable.

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**Option 2 : Nouveau plan App Service**

Ici, nous créons le plan App Service, l’application web et une inscription Bot Channels Registration. 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| Option   | Description |
|:---------|:------------|
| Nom | Nom convivial pour le déploiement. |
| resource-group | Nom du groupe de ressources Azure. |
| template-file | Chemin du modèle ARM. Vous pouvez utiliser le fichier `template-with-preexisting-rg.json` fourni dans le dossier `deploymentTemplates` du projet. |
| location |Lieu. Valeurs provenant de : `az account list-locations`. Vous pouvez configurer le lieu par défaut en utilisant `az configure --defaults location=<location>`. |
| parameters | Spécifiez les valeurs des paramètres du déploiement. Valeur `appId` que vous avez obtenue en exécutant la commande `az ad app create`. `appSecret` est le mot de passe que vous avez fourni à l’étape précédente. Le paramètre `botId`, qui est utilisé comme ID de bot immuable, doit être globalement unique. Il sert aussi à configurer le nom d’affichage du bot, qui est mutable. `newWebAppName` est le nom de l’application web que vous créez. `newAppServicePlanName` est le nom du plan App Service. `newAppServicePlanLocation` est l’emplacement du plan App Service. |

---

### <a name="5-prepare-your-code-for-deployment"></a>5. Préparer votre code en vue du déploiement
#### <a name="51-retrieve-or-create-necessary-iiskudu-files"></a>5.1 Récupérer ou créer les fichiers IIS/Kudu nécessaires

<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Vous devez fournir le chemin du fichier .csproj par rapport à --code-dir. Vous pouvez, pour cela, utiliser l’argument --proj-file-path. La commande résoudrait --code-dir et --proj-file-path en « ./MyBot.csproj ».

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Cette commande récupère un fichier web.config qui est nécessaire au fonctionnement des applications Node.js avec IIS sur Azure App Services. Vérifiez que le fichier web.config est enregistré à la racine de votre bot.

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Cette commande fonctionne de manière similaire à la commande pour JavaScript ci-dessus, mais pour un bot Typescript.

---

> [!NOTE]
> Après l’exécution de la commande mentionnée ci-dessus, vous voyez normalement un fichier `.deployment` dans le dossier du projet de votre bot.

#### <a name="52-zip-up-the-code-directory-manually"></a>5.2 Compresser manuellement le répertoire du code

Quand vous utilisez l’[API de déploiement à partir d’un fichier zip](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) non configurée pour déployer le code de votre bot, le comportement de Kudu/de l’application web est le suivant :

_Par défaut, Kudu part du principe que les déploiements à partir de fichiers zip sont prêts à être exécutés et ne nécessitent aucune étape de génération supplémentaire pendant le déploiement, telle que npm install ou dotnet restore/dotnet publish._

Par conséquent, il est important d’inclure votre code généré et toutes les dépendances nécessaires dans le fichier zip déployé sur l’application web, sans quoi votre bot ne fonctionnera pas comme prévu.

> [!IMPORTANT]
> Avant de compresser vos fichiers projet, vérifiez que vous êtes bien _dans_ le dossier approprié. 
> - Pour les bots C#, il s’agit du dossier contenant le fichier .csproj. 
> - Pour les bots JS, il s’agit du dossier contenant le fichier app.js ou index.js. 
>
>**Dans** le dossier du projet, sélectionnez tous les fichiers et compressez-les, puis exécutez la commande, toujours dans le dossier. 
>
> Si l’emplacement de votre dossier racine est incorrect, l’**exécution du bot échouera dans le portail Azure**.

## <a name="deploy-code-to-azure"></a>Déployer le code sur Azure
À ce stade, nous sommes prêts à déployer le code sur l’application web Azure. Exécutez la commande suivante à partir de la ligne de commande pour effectuer un déploiement push de fichier zip kudu pour une application web.

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Option   | Description |
|:---------|:------------|
| resource-group | Nom du groupe de ressources dans Azure que vous avez créé précédemment. |
| Nom | Nom de l’application web utilisée précédemment. |
| src  | Chemin du fichier compressé que vous avez créé. |

## <a name="test-in-web-chat"></a>Tester dans la Discussion Web

1. Dans votre navigateur, accédez au [Portail Azure](https://ms.portal.azure.com).
2. Dans le panneau de gauche, cliquez sur **Groupes de ressources**.
3. Dans le panneau de droite, recherchez votre groupe.
4. Cliquez sur le nom de votre groupe.
5. Cliquez sur le lien de l’inscription du canal de votre robot.
6. Dans le *panneau d’inscription du canal de robot*, cliquez sur **Tester dans la discussion Web**.
Sinon, dans le panneau de droite, cliquez sur la zone de test.

Pour plus d’informations sur l’inscription du canal, consultez [Inscrire un robot avec Bot Service](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0).

> [!NOTE]
> Un panneau est la surface sur laquelle s’affichent les fonctions de service ou les éléments de navigation lorsqu’ils sont sélectionnés.

## <a name="additional-information"></a>Informations supplémentaires
Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer un déploiement continu](bot-service-build-continuous-deployment.md)
