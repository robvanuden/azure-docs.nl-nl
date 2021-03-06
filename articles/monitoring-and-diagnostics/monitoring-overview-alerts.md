---
title: Overzicht van klassieke waarschuwingen in Microsoft Azure en Azure Monitor | Microsoft Docs
description: Waarschuwingen kunnen u Azure-resource metrische gegevens, gebeurtenissen of Logboeken bewaken en een melding krijgen wanneer een opgegeven voorwaarde wordt voldaan.
author: rboucher
manager: carmonm
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: a6dea224-57bf-43d8-a292-06523037d70b
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/28/2018
ms.author: robb
ms.openlocfilehash: 06ba05f71cf1f696033099c04448526a0421ad42
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/03/2018
---
# <a name="what-are-classic-alerts-in-microsoft-azure"></a>Wat zijn klassieke waarschuwingen in Microsoft Azure?

> [!NOTE]
> In dit artikel wordt beschreven hoe oudere klassieke metrische waarschuwingen maken. Azure ondersteunt van de Monitor nu [nieuwere metrische vrijwel in realtime-waarschuwingen](monitoring-overview-unified-alerts.md)
>

In dit artikel beschrijft de verschillende bronnen van waarschuwingen in Microsoft Azure, wat zijn de doelen voor waarschuwingen, hun voordelen en hoe u aan de slag met het gebruik ervan. Dit geldt met name voor klassieke waarschuwingen van Azure controleren. Waarschuwingen bieden een methode voor het bewaken in Azure die u voorwaarden configureren over gegevens en worden gewaarschuwd kunt wanneer de voorwaarden overeen met de meest recente bewakingsgegevens.


## <a name="taxonomy-of-azure-monitor-classic-alerts"></a>Taxonomie van klassieke waarschuwingen van Azure controleren
Azure maakt gebruik van de volgende termen om klassieke waarschuwingen en hun functies te beschrijven:
* **Waarschuwing** -definitie van een van de criteria (een of meer regels of -voorwaarden) die wordt geactiveerd wanneer voldaan.
* **Actieve** -de status wanneer aan de criteria die zijn gedefinieerd door een klassieke waarschuwing wordt voldaan.
* **Opgelost** -de status wanneer aan de criteria die zijn gedefinieerd door een klassieke waarschuwing niet meer na een eerder zijn voldaan wordt voldaan.
* **Melding** : de actie op die op basis van een klassieke waarschuwing actief.
* **Actie** -specifieke gebeld verzonden naar een ontvanger van een melding (bijvoorbeeld e-mailen een adres of ze naar een webhook-URL). Meldingen kunnen meestal meerdere acties activeren.


## <a name="alerts-on-azure-monitor-data"></a>Waarschuwingen op gegevens van de Azure-Monitor
Er zijn drie soorten waarschuwingen van de gegevens beschikbaar zijn vanuit Azure Monitor--metrische waarschuwingen, bijna realtime metrische waarschuwingen en het activiteitenlogboek van waarschuwingen.

* **Klassieke metrische waarschuwingen** -deze waarschuwing wordt geactiveerd wanneer de waarde van een opgegeven waarde overschrijdt de drempelwaarde die u toewijst. De waarschuwing genereert een melding wanneer de waarschuwing is 'geactiveerd' (als de drempelwaarde is overschreden en de waarschuwing voorwaarde wordt voldaan), evenals wanneer deze 'opgelost is' (wanneer de drempel opnieuw wordt gepasseerd en niet langer aan de voorwaarde wordt voldaan). Dit zijn oudere metrische waarschuwingen. Zie hieronder voor nieuwere metrische waarschuwingen.

* **Near-realtime metrische waarschuwingen** (nieuwere waarschuwing experience) - Dit zijn nieuwere metrische waarschuwingen met verbeterde functies in vergelijking met vorige metrische waarschuwingen worden gegenereerd. Deze waarschuwingen kunnen worden uitgevoerd met een frequentie van 1 min. Ze bieden ook ondersteuning voor bewaking van meerdere (momenteel twee) metrische gegevens.  De waarschuwing genereert een melding wanneer de waarschuwing is 'geactiveerd' (als de drempelwaarden voor elke metriek op hetzelfde moment worden overschreden en de waarschuwing voorwaarde wordt voldaan), evenals wanneer deze 'opgelost is' (wanneer ten minste één metriek overschrijdt de drempelwaarde opnieuw en de voorwaarde is geen meer voldaan aan).

* **Klassieke activiteit logboek waarschuwingen** -een streaming logboek waarschuwing die wordt geactiveerd wanneer een gebeurtenis activiteitenlogboek wordt gegenereerd dat overeenkomt met filtercriteria die u hebt toegewezen. Deze waarschuwingen hebben slechts één status 'Geactiveerd', omdat de waarschuwings-engine gewoon de filtercriteria die van toepassing op een nieuwe gebeurtenis. Deze waarschuwingen kunnen worden gebruikt om te worden geïnformeerd wanneer er een nieuw incident in de servicestatus optreedt of als een gebruiker of toepassing een bewerking uitvoert in uw abonnement, bijvoorbeeld 'Virtuele machine verwijderen'.

Voor diagnostische logboekgegevens die beschikbaar is via Azure Monitor, het is raadzaam de gegevens in logboekanalyse Routering en het gebruik van een Log Analytics-waarschuwing. Het volgende diagram geeft een overzicht van bronnen van de gegevens in Azure Monitor en, conceptueel gezien hoe u waarschuwt af die gegevens.

![Waarschuwingen uitgelegd](./media/monitoring-overview-alerts/Alerts_Overview_Resource_v4.png)

## <a name="how-do-i-receive-a-notification-on-an-azure-monitor-classic-alert"></a>Hoe ontvang ik een melding op een klassieke Azure-monitorwaarschuwing
Waarschuwingen van andere services van Azure gebruikt in het verleden hebben hun eigen ingebouwde notification-methoden. Voortaan kunt biedt Azure een herbruikbare melding groepering actiegroepen genoemd. Actiegroepen Geef een set van ontvangers voor een melding--een aantal e-mailadressen, telefoonnummers (SMS) of webhook-URL's-- en elk gewenst moment een waarschuwing is geactiveerd die verwijst naar de groep actie alle ontvangers die meldingen ontvangen. Hiermee kunt u een groepering van ontvangers (bijvoorbeeld uw op aanroep engineer lijst) over veel waarschuwingen objecten hergebruiken. Op dit moment alleen activiteitenlogboek waarschuwingen maken gebruik van groepen in te grijpen, maar verschillende Waarschuwingstypen van andere Azure werkt ook met behulp van groepen in te grijpen.

Actie beheergroepen ondersteunen melding door ze naar een webhook-URL naast het e-mailadressen en de SMS-nummers. Hiermee kunt automation en het doorvoeren, bijvoorbeeld met behulp van:
    - Azure Automation Runbook
    - Azure-functie
    - Azure Logic App
    - een service van derden

Gebruik actiegroepen bijna realtime metriek waarschuwingen en het activiteitenlogboek van waarschuwingen.

Oudere metrische waarschuwingen onder waarschuwingen (klassiek) gebruik geen actiegroepen. U kunt meldingen configureren op een afzonderlijke waarschuwing metrische:
* E-mailmeldingen verzenden naar de service-beheerder, CO-beheerders of extra e-mailadressen die u opgeeft.
* Aanroepen van een webhook waarmee u kunt aanvullende automation acties te starten.

## <a name="next-steps"></a>Volgende stappen
Informatie ophalen over de regels voor waarschuwingen en ze te configureren met behulp van:

* Meer informatie over [metrische gegevens](monitoring-overview-metrics.md)
* Configureer [klassieke metriek waarschuwingen per Azure-portal](insights-alerts-portal.md)
* Configureer [klassieke metriek waarschuwingen PowerShell](insights-alerts-powershell.md)
* Configureer [klassieke metriek waarschuwingen opdrachtregelinterface (CLI)](insights-alerts-command-line-interface.md)
* Configureer [klassieke metriek waarschuwingen Monitor REST API van Azure](https://msdn.microsoft.com/library/azure/dn931945.aspx)
* Meer informatie over [Activity Log](monitoring-overview-activity-logs.md)
* Configureer [activiteit logboek waarschuwingen per Azure-portal](monitoring-activity-log-alerts.md)
* Configureer [activiteit logboek waarschuwingen via Resource Manager](monitoring-create-activity-log-alerts-with-resource-manager-template.md)
* Controleer de [activiteit logboek waarschuwing webhook schema](monitoring-activity-log-alerts-webhook.md)
* Meer informatie over [nieuwere metriek waarschuwingen](monitoring-near-real-time-metric-alerts.md)
* Meer informatie over [servicemeldingen](monitoring-service-notifications.md)
* Meer informatie over [actiegroepen](monitoring-action-groups.md)
* Configureer [waarschuwingen](monitor-alerts-unified-usage.md)
