---
title: Windows universele Apps SDK-inhoud
description: Meer informatie over de inhoud van de Windows universele Apps SDK voor Azure Mobile Engagement
services: mobile-engagement
documentationcenter: mobile
author: piyushjo
manager: erikre
editor: ''
ms.assetid: 8fa1b701-1c2b-4aec-940c-06c974ef5405
ms.service: mobile-engagement
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-store
ms.devlang: dotnet
ms.topic: article
ms.date: 08/19/2016
ms.author: piyushjo
ms.openlocfilehash: 7b520adcc6af24f7b092580ea82a787a120668bf
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/30/2018
---
# <a name="windows-universal-apps-sdk-content"></a>Windows universele Apps SDK-inhoud
> [!IMPORTANT]
> Azure Mobile Engagement beëindigen op 3/31/2018. Deze pagina wordt kort na worden verwijderd.
> 

Dit document bevat en beschrijft de inhoud die is geïmplementeerd door de SDK in uw toepassing.

## <a name="the-resources-folder"></a>De `/Resources` map
Deze map bevat alle resources die behoeften van de Mobile Engagement. U kunt deze aanpassen aan uw app ook aanpassen.

* `EngagementConfiguration.xml` : De Mobile Engagement-configuratiebestand, dit is waar u de Mobile Engagement-instellingen (Mobile Engagement-verbindingsreeks, rapport crash...) kunt aanpassen.

### <a name="html-folder"></a>de map/HTML
* `EngagementNotification.html` : De `Notification` webontwerp weergave html voor in-app-banner.
* `EngagementAnnouncement.html` : De `Announcement` webontwerp weergave html voor in-app-verspreide weergaven.

### <a name="images-folder"></a>de map/images
* `EngagementIconNotification.png` : Het merk-pictogram weergegeven aan de linkerkant van een melding vervangen deze door uw merk-pictogram.
* `EngagementIconOk.png` : De `Ok` pictogram reach inhoudspagina's voor de knop validatie of actie.
* `EngagementIconNOK.png` : De `NOK` pictogram dat wordt gebruikt wanneer de knop validatie reach inhoudspagina's is uitgeschakeld.
* `EngagementIconClose.png` : De `Close` pictogram van de reach-meldingen en de inhoud voor de knop sluiten.

### <a name="overlay-folder"></a>/overlay map
* `EngagementPageOverlay.cs` : De pagina overlay verantwoordelijk voor het toevoegen van de Engagement-app-gebruikersinterface voor het onderliggende bereiken.

