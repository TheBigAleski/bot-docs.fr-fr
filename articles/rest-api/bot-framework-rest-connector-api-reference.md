---
title: Informations de référence sur l’API | Microsoft Docs
description: Découvrez les en-têtes, les opérations, les objets et les erreurs associés aux services Bot Connector et Bot State.
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/02/2019
ms.openlocfilehash: ea54b9d0d49de972bc8871e18e0bb92c81044514
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037483"
---
# <a name="api-reference"></a>Informations de référence sur l'API

> [!NOTE]
> L’API REST n’est pas équivalente au SDK. L’API REST permet une communication standard REST. Toutefois, la méthode d’interaction recommandée avec Bot Framework reste le SDK. 

Au sein de Bot Framework, le service Bot Connector permet à votre bot d’échanger des messages avec des utilisateurs sur des canaux configurés dans le portail Bot Framework. Le service utilise les ressources REST et JSON standard sur HTTPS.

## <a name="base-uri"></a>URI de base

Lorsqu’un utilisateur envoie un message à votre bot, la requête entrante contient un objet [Activity](#activity-object) avec une propriété `serviceUrl` qui spécifie le point de terminaison auquel votre bot doit envoyer sa réponse. Pour accéder au service Bot Connector, utilisez la valeur `serviceUrl` comme URI de base pour les requêtes d’API.

Supposons, par exemple, que votre bot reçoive l’activité suivante lorsque l’utilisateur lui envoie un message.

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

Dans le message de l’utilisateur, la propriété `serviceUrl` indique que le bot doit envoyer sa réponse au point de terminaison `https://smba.trafficmanager.net/apis`. Il s’agira de l’URI de base pour toutes les prochaines requêtes émises par le bot dans le cadre de cette conversation. Si votre bot a besoin d’envoyer un message proactif à l’utilisateur, veillez à enregistrer la valeur de `serviceUrl`.

L’exemple suivant montre la requête émise par le bot pour répondre au message de l’utilisateur. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>headers

### <a name="request-headers"></a>En-têtes de requête

Outre les en-têtes de requête HTTP standard, chaque requête d’API que vous émettez doit inclure un en-tête `Authorization` qui spécifie un jeton d’accès permettant d’authentifier votre bot. Spécifiez l’en-tête `Authorization` au format suivant :

```http
Authorization: Bearer ACCESS_TOKEN
```

Pour plus d’informations sur l’obtention d’un jeton d’accès pour votre bot, consultez [Authentifier les requêtes envoyées par votre bot au service Bot Connector](bot-framework-rest-connector-authentication.md#bot-to-connector).

### <a name="response-headers"></a>En-têtes de réponse

Outre les en-têtes de réponse HTTP standard, chaque réponse contient un en-tête `X-Correlating-OperationId`. La valeur de cet en-tête est un ID qui correspond à l’entrée de journal Bot Framework où se trouvent des informations détaillées sur la requête. Dès que vous recevez une réponse d’erreur, vous devez capturer la valeur de cet en-tête. Si vous ne parvenez pas à résoudre le problème, fournissez cette valeur à l’équipe du support technique lorsque vous lui signalez le problème.

## <a name="http-status-codes"></a>Codes d’état HTTP

Le <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">code d’état HTTP</a> retourné avec chaque réponse indique le résultat de la requête correspondante. 

| Code d'état HTTP | Signification |
|----|----|
| 200 | La requête a réussi. |
| 201 | La requête a réussi. |
| 202 | La requête a été acceptée en vue de son traitement. |
| 204 | La requête a réussi, mais aucun contenu n’a été retourné. |
| 400 | La requête présentait un format inadéquat ou était incorrecte. |
| 401 | Le bot n’est pas autorisé à envoyer cette requête. |
| 403 | Le bot n’est pas autorisé à effectuer l’opération demandée. |
| 404 | La ressource demandée est introuvable. |
| 500 | Une erreur de serveur interne s’est produite. |
| 503 | Le service n’est pas disponible. |

### <a name="errors"></a>Errors

Toute réponse qui spécifie un code d’état HTTP dans la plage 4xx ou 5xx va inclure un objet [ErrorResponse](#errorresponse-object) dans le corps de la réponse qui fournit des informations sur l’erreur. Si vous recevez une réponse d’erreur dans la plage 4xx, examinez l’objet **ErrorResponse** pour identifier la cause de l’erreur et résoudre votre problème avant de renvoyer la requête.

## <a name="conversation-operations"></a>Opérations de conversation 
Utilisez ces opérations pour créer des conversations, envoyer des messages (activités) et gérer le contenu des conversations.

| Opération | Description |
|----|----|
| [Créer une conversation](#create-conversation) | Crée une conversation. |
| [Envoyer vers la conversation](#send-to-conversation) | Envoie une activité (message) à la fin de la conversation spécifiée. |
| [Répondre à l’activité](#reply-to-activity) | Envoie une activité (message) à la conversation spécifiée, en réponse à l’activité spécifiée. |
| [Obtenir des conversations](#get-conversations) | Obtient la liste des conversations auxquelles un robot a participé. |
| [Obtenir les membres de la conversation](#get-conversation-members) | Obtient les membres de la conversation spécifiée. |
| [Obtenir les membres paginés de la conversation](#get-conversation-paged-members) | Obtient les membres de la conversation spécifiée, une page à la fois. |
| [Obtenir les membres de l’activité](#get-activity-members) | Obtient les membres de l’activité spécifiée dans la conversation spécifiée. |
| [Mettre à jour l’activité](#update-activity) | Met à jour une activité existante. |
| [Supprimer l’activité](#delete-activity) | Supprime une activité existante. |
| [Supprimer un membre de conversation](#delete-conversation-member) | Supprime un membre d’une conversation. |
| [Envoyer l’historique des conversations](#send-conversation-history) | Charge une transcription des activités passées de la conversation. |
| [Charger la pièce jointe vers le canal](#upload-attachment-to-channel) | Charge une pièce jointe directement dans le stockage d’objets blob d’un canal. |

### <a name="create-conversation"></a>Créer une conversation
Crée une conversation.
```http 
POST /v3/conversations
```

| | |
|----|----|
| **Corps de la demande** | Objet `ConversationParameters` |
| **Retourne** | Objet `ConversationResourceResponse` |

### <a name="send-to-conversation"></a>Envoyer vers la conversation
Envoie une activité (message) vers la conversation spécifiée. L’activité est ajoutée à la fin de la conversation, selon l’horodatage ou la sémantique du canal. Pour répondre à un message de la conversation, utilisez plutôt [Répondre à l’activité](#reply-to-activity).
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Corps de la demande** | Objet [Activity](#activity-object) |
| **Retourne** | Objet [Identification](#identification-object) | 

### <a name="reply-to-activity"></a>Répondre à l’activité
Envoie une activité (message) à la conversation spécifiée, en réponse à l’activité spécifiée. L’activité sera ajoutée comme réponse à une autre activité, si le canal le permet. Si le canal ne prend pas en charge les réponses imbriquées, cette opération se comporte comme l’opération [Envoyer vers la conversation](#send-to-conversation).
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Corps de la demande** | Objet [Activity](#activity-object) |
| **Retourne** | Objet [Identification](#identification-object) | 

### <a name="get-conversations"></a>Obtenir des conversations
Obtient la liste des conversations auxquelles un robot a participé.
```http
GET /v3/conversations?continuationToken={continuationToken}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [ConversationResult](#conversationsresult-object) | 

### <a name="get-conversation-members"></a>Obtenir les membres de la conversation
Obtient les membres de la conversation spécifiée.
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Tableau d’objets [ChannelAccount](#channelaccount-object) | 

### <a name="get-conversation-paged-members"></a>Obtenir les membres paginés de la conversation
Obtient les membres de la conversation spécifiée, une page à la fois.
```http
GET /v3/conversations/{conversationId}/pagedmembers?pageSize={pageSize}&continuationToken={continuationToken}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Un tableau d’objets [ChannelAccount](#channelaccount-object) et un jeton de liaison qui peut être utilisé pour obtenir plus de valeurs |

### <a name="get-activity-members"></a>Obtenir les membres de l’activité
Obtient les membres de l’activité spécifiée dans la conversation spécifiée.
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Tableau d’objets [ChannelAccount](#channelaccount-object) | 

### <a name="update-activity"></a>Mettre à jour l’activité
Certains canaux vous permettent de modifier une activité existante afin de refléter le nouvel état d’une conversation de bot. Par exemple, vous pouvez supprimer des boutons dans un message de la conversation après qu’un utilisateur a cliqué sur l’un d’eux. En cas de réussite, cette opération met à jour l’activité spécifiée dans la conversation spécifiée. 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Corps de la demande** | Objet [Activity](#activity-object) |
| **Retourne** | Objet [Identification](#identification-object) | 

### <a name="delete-activity"></a>Supprimer l’activité
Certains canaux vous permettent de supprimer une activité existante. En cas de réussite, cette opération supprime l’activité spécifiée dans la conversation spécifiée.
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Code d’état HTTP indiquant le résultat de l’opération. Rien n’est spécifié dans le corps de la réponse. | 

### <a name="delete-conversation-member"></a>Supprimer un membre de conversation
Supprime un membre d’une conversation. Si ce membre a été le dernier membre de la conversation, la conversation est également supprimée.
```http
DELETE /v3/conversations/{conversationId}/members/{memberId}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Code d’état HTTP indiquant le résultat de l’opération. Rien n’est spécifié dans le corps de la réponse. | 

### <a name="send-conversation-history"></a>Envoyer l’historique des conversations
Charge une transcription des activités passées de la conversation pour permettre au client de les afficher.
```http
POST /v3/conversations/{conversationId}/activities/history
```

| | |
|----|----|
| **Corps de la demande** | Objet [Transcript](#transcript-object). |
| **Retourne** | Objet [ResourceResponse](#resourceresponse-object). | 

### <a name="upload-attachment-to-channel"></a>Charger la pièce jointe vers le canal
Charge une pièce jointe pour la conversation spécifiée, directement dans le stockage d’objets blob d’un canal. Cela permet de stocker des données dans un magasin conforme.
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **Corps de la demande** | Objet [AttachmentUpload](#attachmentupload-object) |
| **Retourne** | Objet [ResourceResponse](#resourceresponse-object). La propriété **id** spécifie l’ID de pièce jointe qui peut être utilisé avec l’opération [Obtenir les informations de la pièce jointe](#get-attachment-info) et l’opération [Obtenir la pièce jointe](#get-attachment). | 

## <a name="attachment-operations"></a>Opérations de pièce jointe 
Utilisez ces opérations pour récupérer des informations sur une pièce jointe, ainsi que les données binaires du fichier.

| Opération | Description |
|----|----|
| [Obtenir les informations de la pièce jointe](#get-attachment-info) | Obtient des informations sur la pièce jointe spécifiée, y compris le nom du fichier, le type du fichier et les affichages disponibles (affichage d’origine ou miniature, par exemple). |
| [Obtenir la pièce jointe](#get-attachment) | Obtient l’affichage spécifié de la pièce jointe spécifiée sous forme de contenu binaire. | 

### <a name="get-attachment-info"></a>Obtenir les informations de la pièce jointe 
Obtient des informations sur la pièce jointe spécifiée, y compris le nom du fichier, son type, et les affichages disponibles (affichage d’origine ou miniature, par exemple).
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [AttachmentInfo](#attachmentinfo-object) | 

### <a name="get-attachment"></a>Obtenir la pièce jointe
Obtient l’affichage spécifié de la pièce jointe spécifiée sous forme de contenu binaire.
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Contenu binaire qui représente l’affichage spécifié de la pièce jointe spécifiée. | 

## <a name="state-operations"></a>Opérations d’état
Le service Microsoft Bot Framework State a été mis hors service le 30 mars 2018. Auparavant, les bots basés sur Azure Bot Service ou le kit SDK Bot Builder avaient une connexion par défaut à ce service hébergé par Microsoft pour stocker les données d’état du bot. Les bots doivent être mis à jour pour utiliser leur propre stockage d’état.

| Opération | Description |
|----|----|
| `Set User Data` | Stocke les données d’état pour un utilisateur d’un canal. |
| `Set Conversation Data` | Stocke les données d’état pour une conversation d’un canal. |
| `Set Private Conversation Data` | Stocke les données d’état d’un utilisateur dans le contexte d’une conversation d’un canal. |
| `Get User Data` | Récupère les données d’état précédemment stockées pour un utilisateur dans toutes les conversations d’un canal. |
| `Get Conversation Data` | Récupère les données d’état précédemment stockées pour une conversation d’un canal. |
| `Get Private Conversation Data` | Récupère les données d’état précédemment stockées pour un utilisateur dans le contexte d’une conversation d’un canal. |
| `Delete State For User` | Supprime les données d’état qui ont été précédemment stockées pour un utilisateur. |

## <a id="objects"></a> Schéma

Le schéma définit l’objet (et ses propriétés) que votre bot peut utiliser pour communiquer avec un utilisateur. 

| Object | Description |
| ---- | ---- |
| [Objet Activity](#activity-object) | Définit un message qui est échangé entre le bot et l’utilisateur. |
| [Objet AnimationCard](#animationcard-object) | Définit une carte pouvant lire des images GIF animées ou de courtes vidéos. |
| [Objet Attachment](#attachment-object) | Définit des informations supplémentaires à inclure dans le message. Une pièce jointe peut être un fichier multimédia (audio, vidéo, image, fichier, etc.) ou une carte enrichie. |
| [Objet AttachmentData](#attachmentdata-object) | Décrit les données d’une pièce jointe. |
| [Objet AttachmentInfo](#attachmentinfo-object) | Décrit une pièce jointe. |
| [Objet AttachmentView](#attachmentview-object) | Définit l’affichage d’une pièce jointe. |
| [Objet AttachmentUpload](#attachmentupload-object) | Définit la pièce jointe à charger. |
| [Objet AudioCard](#audiocard-object) | Définit une carte qui peut lire un fichier audio. |
| [Objet BotData](#botdata-object) | Définit les données d’état pour un utilisateur, pour une conversation, ou pour un utilisateur dans le contexte d’une conversation stockée à l’aide du service Bot State. |
| [Objet CardAction](#cardaction-object) | Définit une action à effectuer. |
| [Objet CardImage](#cardimage-object) | Définit l’image à afficher sur une carte. |
| [Objet ChannelAccount](#channelaccount-object) | Définit le bot ou le compte d’utilisateur sur le canal. |
| [Objet ConversationAccount](#conversationaccount-object) | Définit une conversation sur un canal. |
| [Objet ConversationMembers](#conversationmembers-object) | Définit les membres d’une conversation. |
| [Objet ConversationParameters](#conversationparameters-object) | Définit des paramètres pour la création d’une conversation. |
| [Objet ConversationReference](#conversationreference-object) | Définit un endroit particulier d’une conversation. |
| [Objet ConversationResourceResponse](#conversationresourceresponse-object) | Définit une réponse à [Créer une conversation](#create-conversation). |
| [Objet ConversationResult](#conversationsresult-object) | Définit le résultat d’un appel de [Obtenir des conversations](#get-conversations). |
| [Objet Entity](#entity-object) | Définit un objet d’entité. |
| [Objet Error](#error-object) | Définit une erreur. |
| [Objet ErrorResponse](#errorresponse-object) | Définit une réponse d’API HTTP. |
| [Objet Fact](#fact-object) | Définit une paire clé-valeur contenant un fait. |
| [Objet GeoCoordinates](#geocoordinates-object) | Définit un emplacement géographique à l’aide des coordonnées World Geodetic System (WSG84). |
| [Objet HeroCard](#herocard-object) | Définit une carte avec une grande image, un titre, du texte et des boutons d’action. |
| [Objet Identification](#identification-object) | Identifie une ressource. |
| [Objet MediaEventValue](#mediaeventvalue-object) | Paramètre supplémentaire des événements multimédias. |
| [Objet MediaUrl](#mediaurl-object) | Définit l’URL de la source d’un fichier multimédia. |
| [Objet Mention](#mention-object) | Définit un utilisateur ou un bot mentionné dans la conversation. |
| [Objet MessageReaction](#messagereaction-object) | Définit une réaction à un message. |
| [Objet Place](#place-object) | Définit un lieu mentionné dans la conversation. |
| [Objet ReceiptCard](#receiptcard-object) | Définit une carte qui contient un reçu pour un achat. |
| [Objet ReceiptItem](#receiptitem-object) | Définit une ligne dans un reçu. |
| [Objet ResourceResponse](#resourceresponse-object) | Définit une ressource. |
| [Objet SemanticAction](#semanticaction-object) | Définit une référence à une action de programmation. |
| [Objet SignInCard](#signincard-object) | Définit une carte qui permet à un utilisateur de se connecter à un service. |
| [Objet SuggestedActions](#suggestedactions-object) | Définit les options qu’un utilisateur peut choisir. |
| [Objet ThumbnailCard](#thumbnailcard-object) | Définit une carte avec une miniature, un titre, du texte et des boutons d’action. |
| [Objet ThumbnailUrl](#thumbnailurl-object) | Définit l’URL de la source d’une image. |
| [Objet Transcript](#transcript-object) | Collection d’activités à charger à l’aide de [Envoyer l’historique des conversations](#send-conversation-history). |
| [Objet VideoCard](#videocard-object) | Définit une carte pouvant lire des vidéos. |

### <a name="activity-object"></a>Objet Activity
Définit un message qui est échangé entre le bot et l’utilisateur.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **action** | string | Action à appliquer ou qui a été appliquée. Utilisez la propriété **type** pour déterminer le contexte de l’action. Par exemple, si **type** a la valeur **contactRelationUpdate**, la valeur de la propriété **action** sera **add** si l’utilisateur ajoute votre bot à sa liste de contacts, ou **remove** s’il supprime votre bot de sa liste de contacts. |
| **attachments** | [Attachment](#attachment-object)[] | Tableau d’objets **Attachment** qui définit des informations supplémentaires à inclure dans le message. Chaque pièce jointe peut être soit un fichier multimédia (audio, vidéo, image, fichier, etc.), soit une carte enrichie. |
| **attachmentLayout** | string | Disposition des **pièces jointes** de la carte enrichie incluses dans le message. Peut avoir l’une des valeurs suivantes : **carousel**, **list**. Pour plus d’informations sur les pièces jointes de carte enrichie, consultez [Ajouter des pièces jointes de cartes enrichies aux messages](bot-framework-rest-connector-add-rich-cards.md). |
| **channelData** | objet | Objet contenant le contenu propre au canal. Certains canaux fournissent des fonctionnalités qui nécessitent des informations supplémentaires impossibles à représenter à l’aide du schéma de pièce jointe. Dans ce type de cas, définissez cette propriété sur le contenu propre au canal, tel que défini dans la documentation du canal. Pour plus d’informations, consultez [Implémenter des fonctionnalités spécifiques au canal](bot-framework-rest-connector-channeldata.md). |
| **channelId** | string | ID qui identifie de façon unique le canal. Défini par le canal. | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | Objet **ConversationAccount** qui définit la conversation à laquelle appartient l’activité. |
| **code** | string | Code indiquant pourquoi la conversation s’est terminée. |
| **entities** | object[] | Tableau d’objets qui représente les entités mentionnées dans le message. Les objets de ce tableau peuvent correspondre à n’importe quel objet <a href="http://schema.org/" target="_blank">Schema.org</a>. Par exemple, le tableau peut inclure des objets [Mention](#mention-object) qui identifient une personne mentionnée dans la conversation, ainsi que des objets [Place](#place-object) qui identifient un lieu mentionné dans la conversation. |
| **from** | [ChannelAccount](#channelaccount-object) | Objet **ChannelAccount** qui spécifie l’expéditeur du message. |
| **historyDisclosed** | boolean | Indicateur qui indique si l’historique est divulgué ou non. La valeur par défaut est **false**. |
| **id** | string | ID qui identifie de façon unique l’activité sur le canal. | 
| **inputHint** | string | Valeur qui indique si votre bot accepte, attend ou ignore l’entrée utilisateur, une fois le message remis au client. Peut avoir l’une des valeurs suivantes : **acceptingInput**, **expectingInput**, **ignoringInput**. |
| **locale** | string | Paramètres régionaux de la langue qui doit être utilisée pour afficher du texte dans le message, au format `<language>-<country>`. Le canal utilise cette propriété pour indiquer la langue de l’utilisateur, pour que le bot puisse spécifier des chaînes d’affichage dans cette langue. La valeur par défaut est **en-US**. |
| **localTimestamp** | string | Date et heure auxquelles le message a été envoyé dans le fuseau horaire local, exprimées au format <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | Tableau d’objets **ChannelAccount** qui représente la liste des utilisateurs qui ont rejoint la conversation. Présent uniquement si le **type** d’activité est « conversationUpdate » et si les utilisateurs ont rejoint la conversation. | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | Tableau d’objets **ChannelAccount** qui représente la liste des utilisateurs qui ont quitté la conversation. Présent uniquement si le **type** d’activité est « conversationUpdate » et si les utilisateurs ont quitté la conversation. | 
| **name** | string | Nom de l’opération à appeler ou nom de l’événement. |
| **recipient** | [ChannelAccount](#channelaccount-object) | Objet **ChannelAccount** qui spécifie le destinataire du message. |
| **relatesTo** | [ConversationReference](#conversationreference-object) | Objet **ConversationReference** qui définit un endroit particulier de la conversation. |
| **replyToId** | string | ID du message auquel ce message répond. Pour répondre à un message envoyé par l’utilisateur, définissez cette propriété sur l’ID du message de l’utilisateur. Certains canaux ne prennent pas en charge les réponses à thread. Dans ce cas, le canal ignore cette propriété et utilise une sémantique chronologique (horodatage) pour ajouter le message à la conversation. | 
| **serviceUrl** | string | URL qui spécifie le point de terminaison de service du canal. Défini par le canal. | 
| **speak** | string | Texte que doit prononcer votre bot sur un canal où les fonctionnalités vocales sont activées. Pour contrôler les différentes caractéristiques vocales de votre bot, telles que la voix, le débit, le volume, la prononciation et la hauteur, spécifiez cette propriété au format <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">SSML (Speech Synthesis Markup Language)</a>. |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | Objet **SuggestedActions** qui définit les options qu’un utilisateur peut choisir. |
| **summary** | string | Résumé des informations que contient le message. Par exemple, pour un message envoyé sur un canal d’e-mail, cette propriété peut spécifier les 50 premiers caractères de l’e-mail. |
| **text** | string | Texte du message envoyé par l’utilisateur au bot ou par le bot à l’utilisateur. Consultez la documentation du canal pour connaître les limites qui sont imposées au contenu de cette propriété. |
| **textFormat** | string | Format du **texte** du message. Peut avoir l’une des valeurs suivantes : **markdown**, **plain**, **xml**. Pour plus d’informations sur le format du texte, consultez [Créer des messages](bot-framework-rest-connector-create-messages.md). |
| **timestamp** | string | Date et heure auxquelles le message a été envoyé dans le fuseau horaire UTC, exprimées au format <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. |
| **topicName** | string | Sujet de la conversation à laquelle appartient l’activité. |
| **type** | string | Type de l’activité. Peut prendre l’une des valeurs suivantes : **contactRelationUpdate**, **conversationUpdate**, **deleteUserData**, **message**, **typing**, **event** et **endOfConversation**. Pour plus d’informations sur les types d’activités, consultez [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md). |
| **value** | objet | Valeur à durée indéterminée. |
| **semanticAction** |[SemanticAction](#semanticaction-object) | Un objet **SemanticAction** qui représente une référence à une action de programmation. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="animationcard-object"></a>Objet AnimationCard
Définit une carte pouvant lire des images GIF animées ou de courtes vidéos.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **autoloop** | boolean | Indicateur qui indique s’il faut relire la liste des images GIF animées lorsque la dernière est terminée. Définissez cette propriété sur **true** pour relire automatiquement l’animation ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **autostart** | boolean | Indicateur qui indique s’il faut lire automatiquement l’animation lorsque la carte s’affiche. Définissez cette propriété sur **true** pour lire automatiquement l’animation ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **duration** | string | La longueur du contenu du média, au [format de durée ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Objet **ThumbnailUrl** qui spécifie l’image à afficher sur la carte. |
| **media** | [MediaUrl](#mediaurl-object)[] | Tableau d’objets **MediaUrl** qui spécifie la liste des images GIF animées à lire. |
| **shareable** | boolean | Indicateur qui indique si l’animation peut être partagée avec d’autres utilisateurs. Définissez cette propriété sur **true** si l’animation peut être partagée ; sinon, sur **false**. La valeur par défaut est **true**. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la carte. |
| **text** | string | Description ou invitation à afficher sous le titre ou le sous-titre de la carte. |
| **title** | string | Titre de la carte. |
| **value** | objet | Paramètre supplémentaire de cette carte |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="attachment-object"></a>Objet Attachment
Définit des informations supplémentaires à inclure dans le message. Une pièce jointe peut être un fichier multimédia (audio, vidéo, image, fichier, etc.) ou une carte enrichie.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **contentType** | string | Type de média du contenu de la pièce jointe. Pour les fichiers multimédias, définissez cette propriété sur les types de médias connus comme **image/png**, **audio/wav** ou **video/mp4**. Pour les cartes enrichies, définissez cette propriété sur l’un de ces types spécifiques au fabricant :<ul><li>**application/vnd.microsoft.card.adaptive** : carte enrichie pouvant inclure toute combinaison de texte, données vocales, images, boutons et champs d’entrée. Définissez la propriété **content** sur un objet <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a>.</li><li>**application/vnd.microsoft.card.animation** : carte enrichie qui lit l’animation. Définissez la propriété **content** sur un objet [AnimationCard](#animationcard-object).</li><li>**application/vnd.microsoft.card.audio** : carte enrichie qui lit les fichiers audio. Définissez la propriété **content** sur un objet [AudioCard](#audiocard-object).</li><li>**application/vnd.microsoft.card.video** : carte enrichie qui lit les vidéos. Définissez la propriété **content** sur un objet [VideoCard](#videocard-object).</li><li>**application/vnd.microsoft.card.hero** : carte Hero. Définissez la propriété **content** sur un objet [HeroCard](#herocard-object).</li><li>**application/vnd.microsoft.card.thumbnail** : carte de miniature. Définissez la propriété **content** sur un objet [ThumbnailCard](#thumbnailcard-object).</li><li>**application/vnd.microsoft.com.card.receipt** : carte de reçu. Définissez la propriété **content** sur un objet [ReceiptCard](#receiptcard-object).</li><li>**application/vnd.microsoft.com.card.signin** : carte de connexion utilisateur. Définissez la propriété **content** sur un objet [SignInCard](#signincard-object).</li></ul> |
| **contentUrl** | string | URL du contenu de la pièce jointe. Par exemple, si la pièce jointe est une image, définissez **contentUrl** sur l’URL qui représente l’emplacement de l’image. Protocoles pris en charge : HTTP, HTTPS, File et Data. |
| **content** | objet | Contenu de la pièce jointe. Si la pièce jointe est une carte enrichie, définissez cette propriété sur l’objet de carte enrichie. Cette propriété et la propriété **contentUrl** s’excluent mutuellement. |
| **name** | string | Nom de la pièce jointe. |
| **thumbnailUrl** | string | URL d’une image miniature que le canal peut utiliser s’il prend en charge l’utilisation d’une forme plus petite de **content** ou de **contentUrl**. Par exemple, si vous définissez **contentType** sur **application/word** et définissez **contentUrl** sur l’emplacement du document Word, vous pouvez inclure une image miniature qui représente le document. Le canal peut alors afficher l’image miniature au lieu du document. Lorsque l’utilisateur clique sur l’image, le canal ouvre le document. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="attachmentdata-object"></a>Objet AttachmentData 
Décrit les données d’une pièce jointe.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **name** | string | Nom de la pièce jointe. |
| **originalBase64** | string | Contenu de la pièce jointe. |
| **thumbnailBase64** | string | Contenu de la miniature de la pièce jointe. |
| **type** | string | Type de contenu de la pièce jointe. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="attachmentinfo-object"></a>Objet AttachmentInfo
Décrit une pièce jointe.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **name** | string | Nom de la pièce jointe. |
| **type** | string | Type de contenu de la pièce jointe. |
| **vues** | [AttachmentView](#attachmentview-object)[] | Tableau d’objets **AttachmentView** qui représentent les affichages disponibles pour la pièce jointe. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="attachmentview-object"></a>Objet AttachmentView
Définit l’affichage d’une pièce jointe.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **viewId** | string | ID de l’affichage. |
| **taille** | number | Taille du fichier. |

<a href="#objects">Retour au tableau Schéma</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>Objet AttachmentUpload
Définit la pièce jointe à charger.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **type** | string | Type de contenu de la pièce jointe. | 
| **name** | string | Nom de la pièce jointe. | 
| **originalBase64** | string | Données binaires qui représentent le contenu de la version d’origine du fichier. |
| **thumbnailBase64** | string | Données binaires qui représentent le contenu de la version miniature du fichier. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="audiocard-object"></a>Objet AudioCard
Définit une carte qui peut lire un fichier audio.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **aspect** | string | Proportions de la miniature qui est spécifiée dans la propriété **image**. Les valeurs valides sont **16:9** et **9:16**. |
| **autoloop** | boolean | Indicateur qui indique s’il faut relire la liste des fichiers audio lorsque le dernier est terminé. Définissez cette propriété sur **true** pour relire automatiquement les fichiers audio ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **autostart** | boolean | Indicateur qui indique s’il faut lire automatiquement le fichier audio lorsque la carte s’affiche. Définissez cette propriété sur **true** pour relire automatiquement les fichiers audio ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **duration** | string | La longueur du contenu du média, au [format de durée ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Objet **ThumbnailUrl** qui spécifie l’image à afficher sur la carte. |
| **media** | [MediaUrl](#mediaurl-object)[] | Tableau d’objets **MediaUrl** qui spécifie la liste des fichiers audio à lire. |
| **shareable** | boolean | Indicateur qui indique si les fichiers audio peuvent être partagés avec d’autres utilisateurs. Définissez cette propriété sur **true** si les fichiers audio peuvent être partagés ; sinon, sur **false**. La valeur par défaut est **true**. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la carte. |
| **text** | string | Description ou invitation à afficher sous le titre ou le sous-titre de la carte. |
| **title** | string | Titre de la carte. |
| **value** | objet | Paramètre supplémentaire de cette carte |

<a href="#objects">Retour au tableau Schéma</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>Objet BotData
Définit les données d’état pour un utilisateur, pour une conversation, ou pour un utilisateur dans le contexte d’une conversation stockée à l’aide du service Bot State.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **data** | objet | Dans une requête, objet JSON qui spécifie les propriétés et les valeurs à stocker à l’aide du service Bot State. Dans une réponse, objet JSON qui spécifie les propriétés et les valeurs qui ont été stockées à l’aide du service Bot State. | 
| **eTag** | string | Valeur de l’étiquette d’entité que vous pouvez utiliser pour contrôler l’accès concurrentiel aux données, pour les données que vous stockez à l’aide du service Bot State. Pour plus d’informations, consultez [Gérer les données d’état](bot-framework-rest-state.md). | 

<a href="#objects">Retour au tableau Schéma</a>

### <a name="cardaction-object"></a>Objet CardAction
Définit une action à effectuer.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **image** | string | URL de l’image à afficher | 
| **text** | string | Texte de l’action |
| **title** | string | Texte du bouton. Applicable uniquement à l’action d’un bouton. |
. Applicable uniquement à l’action d’un bouton. |
| **type** | string | Type d’action à effectuer. Pour obtenir la liste des valeurs valides, consultez [Ajouter des pièces jointes de cartes riches aux messages](bot-framework-rest-connector-add-rich-cards.md). |
| **value** | objet | Paramètre supplémentaire de l’action. La valeur de cette propriété varie en fonction du **type** de l’action. Pour plus d’informations, consultez [Ajouter des pièces jointes de cartes enrichies aux messages](bot-framework-rest-connector-add-rich-cards.md). |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="cardimage-object"></a>Objet CardImage
Définit l’image à afficher sur une carte.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **alt** | string | Description de l’image. Vous devez inclure la description pour des raisons d’accessibilité. |
| **tap** | [CardAction](#cardaction-object) | Objet **CardAction** qui spécifie l’action à effectuer si l’utilisateur appuie ou clique sur l’image. |
| **url** | string | URL de la source de l’image ou du fichier binaire en base64 de l’image (par exemple, `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="channelaccount-object"></a>Objet ChannelAccount
Définit le bot ou le compte d’utilisateur sur le canal.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **id** | string | ID unique de l’utilisateur ou du robot sur ce canal. |
| **name** | string | Nom convivial du robot ou de l’utilisateur. |
| **aadObjectId** | string | L’ID d’objet de ce compte dans Azure Active Directory. |
| **role** | énumération de chaînes | Rôle de l’entité derrière le compte. `user` ou `bot`. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationaccount-object"></a>Objet ConversationAccount
Définit une conversation sur un canal.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **id** | string | ID qui identifie la conversation. L’ID est unique sur chaque canal. Si le canal lance la conversion, il définit cet ID. Sinon, le bot définit cette propriété sur l’ID qu’il reçoit dans la réponse lorsqu’il démarre la conversation (voir Démarrage d’une conversation). |
| **isGroup** | boolean | Indicateur qui indique si la conversation contient plus de deux participants au moment où l’activité est générée. Définissez la valeur **true** s’il s’agit d’une conversation de groupe ; sinon, **false**. La valeur par défaut est **false**. |
| **name** | string | Nom convivial qui peut être utilisé pour identifier la conversation. |
| **conversationType** | string | Indique le type de la conversation sur les canaux qui distinguent les différents types de conversations (par exemple : personnelle, de groupe). |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationmembers-object"></a>Objet ConversationMembers
Définit les membres d’une conversation.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **id** | string | ID de conversation. |
| **members** | array | Tableau d’objets [ChannelAccount](#channelaccount-object). |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationparameters-object"></a>Objet ConversationParameters
Définit des paramètres pour la création d’une conversation.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **isGroup** | boolean | Indique s’il s’agit d’une conversation de groupe. |
| **bot** | [ChannelAccount](#channelaccount-object) | Informations de compte de canal nécessaires à l’acheminement d’un message vers le robot. |
| **members** | Tableau [ChannelAccount](#channelaccount-object) | Informations de compte de canal nécessaires à l’acheminement d’un message vers chaque utilisateur. |
| **topicName** | string | Facultatif, rubrique de la conversation. Cette propriété est uniquement utilisée si le canal la prend en charge. |
| **tennantId** | string | Facultatif, l’ID de l’abonné dans lequel la conversation doit être créée. |
| **activity** | [Activité](#activity-object) | Facultatif, le message initial à envoyer vers la conversation lorsqu’elle est créée. |
| **channelData** | objet | Charge utile spécifique au canal pour la création de la conversation. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationreference-object"></a>Objet ConversationReference
Définit un endroit particulier d’une conversation.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **activityId** | string | ID qui identifie de façon unique l’activité que cet objet référence. | 
| **bot** | [ChannelAccount](#channelaccount-object) | Objet **ChannelAccount** qui identifie le bot dans la conversation référencée par cet objet. |
| **channelId** | string | ID qui identifie de façon unique le canal dans la conversation référencée par cet objet. | 
| **conversation** | [ConversationAccount](#conversationaccount-object) | Objet **ConversationAccount** qui définit la conversation référencée par cet objet. |
| **serviceUrl** | string | URL qui spécifie le point de terminaison de service du canal dans la conversation référencée par cet objet. | 
| **user** | [ChannelAccount](#channelaccount-object) | Objet **ChannelAccount** qui identifie l’utilisateur dans la conversation référencée par cet objet. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationresourceresponse-object"></a>Objet ConversationResourceResponse
Définit une réponse à [Créer une conversation](#create-conversation).<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **activityId** | string | ID de l’activité, si elle est envoyée. |
| **id** | string | ID de la ressource. |
| **serviceUrl** | string | Point de terminaison de service dans lequel les opérations relatives à la conversation peuvent être effectuées. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="conversationsresult-object"></a>Objet ConversationResult
Définit le résultat de [Obtenir des conversations](#get-conversations).<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **continuationToken** | string | Jeton de continuation qui peut être utilisé dans les appels postérieurs à l’appel de [Obtenir des conversations](#get-conversations). |
| **conversations** | array | Tableau d’objets [ConversationMembers](#conversationmembers-object) |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="error-object"></a>Objet Error
Définit une erreur.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **code** | string | Code d’erreur. |
| **message** | string | Description de l’erreur. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="entity-object"></a>Objet Entity
Définit un objet d’entité.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **type** | string | Type d’entité. En général, contient des types de schema.org. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="errorresponse-object"></a>Objet ErrorResponse
Définit une réponse d’API HTTP.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **error** | [Error](#error-object) | Objet **Error** contenant des informations concernant l’erreur. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="fact-object"></a>Objet Fact
Définit une paire clé-valeur contenant un fait.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **key** | string | Nom du fait. Par exemple : **Archivage**. La clé est utilisée comme étiquette quand vous affichez la valeur du fait. |
| **value** | string | Valeur du fait. Par exemple, **10 octobre 2016**. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="geocoordinates-object"></a>Objet GeoCoordinates
Définit un emplacement géographique à l’aide des coordonnées World Geodetic System (WSG84).<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **elevation** | number | Élévation de l’emplacement. |
| **name** | string | Nom de l’emplacement. |
| **latitude** | number | Latitude de l’emplacement. |
| **longitude** | number | Longitude de l’emplacement. |
| **type** | string | Type de cet objet. Toujours définir sur **GeoCoordinates**. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="herocard-object"></a>Objet HeroCard
Définit une carte avec une grande image, un titre, du texte et des boutons d’action.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **images** | [CardImage](#cardimage-object)[] | Tableau d’objets **CardImage** qui spécifie l’image à afficher sur la carte. Une carte Héros contient une seule image. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la carte. |
| **tap** | [CardAction](#cardaction-object) | Objet **CardAction** qui spécifie l’action à effectuer si l’utilisateur appuie ou clique sur la carte. Il peut s’agir de la même action que celle d’un bouton, ou d’une action différente. |
| **text** | string | Description ou invitation à afficher sous le titre ou le sous-titre de la carte. |
| **title** | string | Titre de la carte. |

<a href="#objects">Retour au tableau Schéma</a>

<!--TODO can't find-->
### <a name="identification-object"></a>Objet Identification
Identifie une ressource.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **id** | string | ID qui identifie de façon unique la ressource. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="mediaeventvalue-object"></a>Objet MediaEventValue 
Paramètre supplémentaire des événements multimédias.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **cardValue** | objet | Paramètre de rappel spécifié dans le champ **Valeur** de la carte média qui a déclenché cet événement. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="mediaurl-object"></a>Objet MediaUrl
Définit l’URL de la source d’un fichier multimédia.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **profile** | string | Indicateur qui décrit le contenu du fichier multimédia. |
| **url** | string | URL de la source du fichier multimédia. |

<a href="#objects">Retour au tableau Schéma</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Objet Mention
Définit un utilisateur ou un bot mentionné dans la conversation.<br/><br/> 


|          Propriété          |                   Type                   |                                                                                                                                                                                                                           Description                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>mentioned</strong> | [ChannelAccount](#channelaccount-object) | Objet <strong>ChannelAccount</strong> qui spécifie l’utilisateur ou le bot mentionné. Notez que certains canaux, tels que Slack, attribuent des noms pour chaque conversation. Il est donc possible que le nom mentionné de votre bot (dans la propriété <strong>recipient</strong> du message) soit différent du handle que vous avez spécifié lorsque vous avez [inscrit](../bot-service-quickstart-registration.md) votre bot. Toutefois, l’ID de compte des deux noms reste le même. |
|   <strong>text</strong>    |                  string                  |                                                                                                                         Utilisateur ou bot mentionné dans la conversation. Par exemple, si le message est « @ColorBot, choisis une nouvelle couleur », cette propriété est définie sur <strong>@ColorBot</strong>. Certains canaux ne définissent pas cette propriété.                                                                                                                          |
|   <strong>type</strong>    |                  string                  |                                                                                                                                                                                                   Type de cet objet. Toujours définir sur <strong>Mention</strong>.                                                                                                                                                                                                    |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="messagereaction-object"></a>Objet MessageReaction
Définit une réaction à un message.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **type** | string | Type de réaction. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="place-object"></a>Objet Place
Définit un lieu mentionné dans la conversation.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **address** | objet |  Adresse d’un lieu. Cette propriété peut être un `string` ou un objet complexe de type `PostalAddress`. |
| **geo** | [GeoCoordinates](#geocoordinates-object) | Objet **GeoCoordinates** qui spécifie les coordonnées géographiques du lieu. |
| **hasMap** | objet | Carte du lieu. Cette propriété peut être un `string` (URL) ou un objet complexe de type `Map`. |
| **name** | string | Nom du lieu. |
| **type** | string | Type de cet objet. Toujours définir sur **Place**. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="receiptcard-object"></a>Objet ReceiptCard
Définit une carte qui contient un reçu pour un achat.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **facts** | [Fact](#fact-object)[] | Tableau d’objets **Fact** qui spécifient des informations sur l’achat. Par exemple, la liste de faits du reçu d’un séjour à l’hôtel peut inclure les dates d’arrivée et de départ. Le canal détermine le nombre de faits que vous pouvez spécifier. |
| **items** | [ReceiptItem](#receiptitem-object)[] | Tableau d’objets **ReceiptItem** qui spécifient les articles achetés |
| **tap** | [CardAction](#cardaction-object) | Objet **CardAction** qui spécifie l’action à effectuer si l’utilisateur appuie ou clique sur la carte. Il peut s’agir de la même action que celle d’un bouton, ou d’une action différente. |
| **tax** | string | Chaîne au format devise qui spécifie la somme de la taxe appliquée à l’achat. |
| **title** | string | Titre affiché en haut du reçu. |
| **total** | string | Chaîne au format devise qui spécifie le prix total d’un achat, y compris toutes les taxes applicables. |
| **vat** | string | Chaîne au format devise qui spécifie la somme de la TVA appliquée au prix de l’achat. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="receiptitem-object"></a>Objet ReceiptItem
Définit une ligne dans un reçu.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **image** | [CardImage](#cardimage-object) | Objet **CardImage** qui spécifie l’image miniature à afficher à côté de la ligne.  |
| **price** | string | Chaîne au format devise qui spécifie le prix total de toutes les unités achetées. |
| **quantity** | string | Chaîne numérique qui spécifie le nombre d’unités achetées. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la ligne. |
| **tap** | [CardAction](#cardaction-object) | Objet **CardAction** qui spécifie l’action à effectuer si l’utilisateur appuie ou clique sur la ligne. |
| **text** | string | Description de la ligne. |
| **title** | string | Titre de la ligne. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="resourceresponse-object"></a>Objet ResourceResponse
Définit une réponse qui contient un ID de ressource.<br/><br/>

|      Propriété       |  Type  |                Description                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | string | ID qui identifie de façon unique la ressource. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="semanticaction-object"></a>Objet SemanticAction
Définit une référence à une action de programmation.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **id** | string | ID de cette action |
| **entities** | [Entité](#entity-object) | Entités associées à cette action |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="signincard-object"></a>Objet SignInCard
Définit une carte qui permet à un utilisateur de se connecter à un service.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur de se connecter à un service. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **text** | string | Description ou invitation à inclure dans la carte de connexion. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="suggestedactions-object"></a>Objet SuggestedActions
Définit les options qu’un utilisateur peut choisir.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui définissent les actions suggérées. |
| **to** | string[] | Tableau de chaînes qui contient les ID des destinataires pour lesquels les actions suggérées doivent être affichées. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="thumbnailcard-object"></a>Objet ThumbnailCard
Définit une carte avec une miniature, un titre, du texte et des boutons d’action.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **images** | [CardImage](#cardimage-object)[] | Tableau d’objets **CardImage** qui spécifie les images miniatures à afficher sur la carte. Le canal détermine le nombre d’images miniatures que vous pouvez spécifier. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la carte. |
| **tap** | [CardAction](#cardaction-object) | Objet **CardAction** qui spécifie l’action à effectuer si l’utilisateur appuie ou clique sur la carte. Il peut s’agir de la même action que celle d’un bouton, ou d’une action différente. |
| **text** | string | Description ou invitation à afficher sous le titre ou le sous-titre de la carte. |
| **title** | string | Titre de la carte. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="thumbnailurl-object"></a>Objet ThumbnailUrl
Définit l’URL de la source d’une image.<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **alt** | string | Description de l’image. Vous devez inclure la description pour des raisons d’accessibilité. |
| **url** | string | URL de la source de l’image ou du fichier binaire en base64 de l’image (par exemple, `data:image/png;base64,iVBORw0KGgo...`). |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="transcript-object"></a>Objet Transcript
Collection d’activités à charger à l’aide de [Envoyer l’historique des conversations](#send-conversation-history).<br/><br/> 

| Propriété | Type | Description |
|----|----|----|
| **activités** | array | Tableau d’objets [Activity](#activity-object). Ils doivent posséder chacun un ID unique et un horodatage. |

<a href="#objects">Retour au tableau Schéma</a>

### <a name="videocard-object"></a>Objet VideoCard
Définit une carte pouvant lire des vidéos.<br/><br/>

| Propriété | Type | Description |
|----|----|----|
| **aspect** | string | Proportions d’une vidéo (par exemple, 16:9, 4:3).|
| **autoloop** | boolean | Indicateur qui indique s’il faut relire la liste des vidéos lorsque la dernière est terminée. Définissez cette propriété sur **true** pour relire automatiquement les vidéos ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **autostart** | boolean | Indicateur qui indique s’il faut lire automatiquement les vidéos lorsque la carte s’affiche. Définissez cette propriété sur **true** pour lire automatiquement les vidéos ; sinon, définissez-la sur **false**. La valeur par défaut est **true**. |
| **buttons** | [CardAction](#cardaction-object)[] | Tableau d’objets **CardAction** qui permettent à l’utilisateur d’effectuer une ou plusieurs actions. Le canal détermine le nombre de boutons que vous pouvez spécifier. |
| **duration** | string | La longueur du contenu du média, au [format de durée ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). |
| **image** | [ThumbnailUrl](#thumbnailurl-object) | Objet **ThumbnailUrl** qui spécifie l’image à afficher sur la carte. |
| **media** | [MediaUrl](#mediaurl-object)[] | Tableau d’objets **MediaUrl** qui spécifie la liste des vidéos à lire. |
| **shareable** | boolean | Indicateur qui indique si les vidéos peuvent être partagées avec d’autres utilisateurs. Définissez cette propriété sur **true** si les vidéos peuvent être partagées ; sinon, sur **false**. La valeur par défaut est **true**. |
| **subtitle** | string | Sous-titre à afficher sous le titre de la carte. |
| **text** | string | Description ou invitation à afficher sous le titre ou le sous-titre de la carte. |
| **title** | string | Titre de la carte. |
| **value** | objet | Paramètre supplémentaire de cette carte|

<a href="#objects">Retour au tableau Schéma</a>
