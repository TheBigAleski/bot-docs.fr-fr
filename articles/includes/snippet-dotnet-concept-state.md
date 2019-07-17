---
ms.openlocfilehash: 0b991c438c0006d1fb4bafa90982f73f4a18be77
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230663"
---
L’infrastructure Bot Builder permet à votre bot de stocker et de récupérer les données d’état qui sont associées à un utilisateur, à une conversation ou à un utilisateur spécifique dans le contexte d’une conversation donnée. Les données d’état peuvent servir à de nombreuses fins, par exemple pour déterminer l’endroit où une conversation précédente s’était arrêtée ou simplement pour accueillir un utilisateur régulier par son nom. Si vous stockez les préférences d’un utilisateur, vous pouvez utiliser ces informations pour personnaliser la prochaine conversation. Par exemple, vous pourriez avertir l’utilisateur de la publication d’un article sur un sujet qui l’intéresse ou bien l’informer de la disponibilité d’une date de rendez-vous. 

À des fins de test et de prototypage, vous pouvez utiliser le stockage de données en mémoire de l’infrastructure Bot Builder. Pour les bots de production, vous pouvez implémenter votre propre adaptateur de stockage ou utiliser l’une des Extensions Azure. Les Extensions Azure permettent de stocker les données d’état de votre bot dans Stockage Table, CosmosDB ou SQL. Cet article montre comment utiliser l’adaptateur de stockage en mémoire pour stocker les données d’état de votre bot. 

> [!IMPORTANT]
> L’API du service Bot Framework State n’est pas recommandée pour les environnements de production et peut être déconseillée dans une version ultérieure. Nous vous recommandons de mettre à jour le code de votre bot pour qu’il utilise l’adaptateur de stockage en mémoire à des fins de test ou pour qu’il utilise l’une des **extensions Azure** pour les bots de production.
