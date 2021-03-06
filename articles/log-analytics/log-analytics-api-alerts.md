---
title: Met behulp van OMS Log Analytics waarschuwing REST-API
description: De waarschuwing REST-API van Log Analytics kunt u waarschuwingen in logboekanalyse die deel uitmaakt van de Operations Management Suite (OMS) maken en beheren.  Dit artikel bevat de details van de API en enkele voorbeelden voor het uitvoeren van verschillende bewerkingen.
services: log-analytics
documentationcenter: ''
author: bwren
manager: jwhit
editor: tysonn
ms.assetid: 628ad256-7181-4a0d-9e68-4ed60c0f3f04
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/10/2018
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 642b27405c703aa7a30d9fc544009d70b5d1b2df
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="create-and-manage-alert-rules-in-log-analytics-with-rest-api"></a>Maken en beheren van waarschuwingsregels in logboekanalyse met REST-API
De waarschuwing REST-API van Log Analytics kunt u waarschuwingen in Operations Management Suite (OMS) maken en beheren.  Dit artikel bevat de details van de API en enkele voorbeelden voor het uitvoeren van verschillende bewerkingen.

Log Analytics Search REST-API RESTful is en toegankelijk zijn via de REST-API van Azure Resource Manager. In dit document vindt u voorbeelden waarbij de API worden geopend vanuit een PowerShell-opdrachtregel met [ARMClient](https://github.com/projectkudu/ARMClient), een open source-opdrachtregelprogramma dat vereenvoudigt de Azure Resource Manager-API wordt aangeroepen. Het gebruik van ARMClient en PowerShell is een van de vele opties voor toegang tot de Log Analytics zoeken-API. Met deze hulpprogramma's, kunt u gebruikmaken van de RESTful Azure Resource Manager-API aanroepen met OMS-werkruimten en uitvoeren van opdrachten in deze zoekopdracht. De API uitvoer zoekresultaten aan u in de JSON-indeling, zodat u kunt de lijst met zoekresultaten programmatisch op veel verschillende manieren gebruiken.

## <a name="prerequisites"></a>Vereisten
Waarschuwingen kunnen op dit moment kunnen alleen worden gemaakt met een opgeslagen zoekopdracht in logboekanalyse.  U kunt verwijzen naar de [logboek Search REST-API](log-analytics-log-search-api.md) voor meer informatie.

## <a name="schedules"></a>Planningen
Een opgeslagen zoekopdracht kan een of meer planningen hebben. Het schema wordt gedefinieerd hoe vaak de zoekopdracht is uitgevoerd en de tijdsinterval waarover de criteria wordt geïdentificeerd.
Schema's hebben de eigenschappen in de volgende tabel.

| Eigenschap | Beschrijving |
|:--- |:--- |
| Interval |Hoe vaak de zoekactie wordt uitgevoerd. Gemeten in minuten. |
| QueryTimeSpan |Het interval waarmee de criteria wordt geëvalueerd. Moet gelijk zijn aan of groter dan het Interval. Gemeten in minuten. |
| Versie |De API-versie wordt gebruikt.  Op dit moment is moet dit altijd worden ingesteld op 1. |

Neem bijvoorbeeld een event-query met een Interval van 15 minuten en een TimeSpan-waarde van 30 minuten. In dit geval wordt de query wilt uitvoeren om de 15 minuten en een waarschuwing zou worden geactiveerd als de criteria nog steeds omzetten in true over een reeks van 30 minuten.

### <a name="retrieving-schedules"></a>Bij het ophalen van schema 's
Gebruik de Get-methode voor het ophalen van alle schema's voor een opgeslagen zoekopdracht.

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules?api-version=2015-03-20

Gebruik de Get-methode met een schema-ID voor het ophalen van een bepaalde planning voor een opgeslagen zoekopdracht.

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20

Hier volgt een voorbeeldantwoord voor een schema.

```json
{
    "value": [{
        "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace/savedSearches/0f0f4853-17f8-4ed1-9a03-8e888b0d16ec/schedules/a17b53ef-bd70-4ca4-9ead-83b00f2024a8",
        "etag": "W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\"",
        "properties": {
            "Interval": 15,
            "QueryTimeSpan": 15
        }
    }]
}
```

### <a name="creating-a-schedule"></a>Een schema maken
De Put-methode met een uniek schema-ID gebruiken om een nieuwe planning te maken.  Houd er rekening mee dat twee planningen kunnen niet dezelfde ID zelfs als ze gekoppeld aan verschillende zijn opgeslagen zoekopdrachten.  Wanneer u een planning in de OMS-console maakt, wordt een GUID gemaakt voor de schema-ID.

> [!NOTE]
> De naam op voor alle opgeslagen zoekopdrachten, schema's en acties die zijn gemaakt met de Log Analytics-API moet in kleine letters.

    $scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson

### <a name="editing-a-schedule"></a>Het bewerken van een planning
Gebruik de Put-methode met een bestaand schema-ID voor de dezelfde opgeslagen zoekopdracht om te wijzigen die planning.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de planning.

      $scheduleJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\""','properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' } }"
      armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson


### <a name="deleting-schedules"></a>Schema's verwijderen
Gebruik de Delete-methode met een schema-ID om te verwijderen van een planning.

    armclient delete /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20


## <a name="actions"></a>Acties
Een planning kan meerdere acties hebben. Een actie kan definiëren een of meer processen uit te voeren zoals het verzenden van een e-mail of een runbook starten, of het kan een drempelwaarde die bepaalt wanneer de resultaten van een zoekopdracht voldoen aan bepaalde criteria definiëren.  Sommige acties definiëren beide zodat de processen worden uitgevoerd wanneer de drempelwaarde wordt voldaan.

Alle acties hebben de eigenschappen in de volgende tabel.  Verschillende soorten waarschuwingen hebben verschillende aanvullende eigenschappen, die hieronder worden beschreven.

| Eigenschap | Beschrijving |
|:--- |:--- |
| Type |Type van de actie.  De mogelijke waarden zijn momenteel waarschuwing en Webhook. |
| Naam |Weergavenaam voor de waarschuwing. |
| Versie |De API-versie wordt gebruikt.  Op dit moment is moet dit altijd worden ingesteld op 1. |

### <a name="retrieving-actions"></a>Bij het ophalen van acties

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md). Acties zijn voor gebruikers die waarschuwingen naar Azure uitbreiden, nu beheerd met Azure Actiegroepen. Wanneer een werkruimte en de waarschuwingen worden uitgebreid naar Azure, kunt u ophalen of acties toevoegen met behulp van de [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).

Gebruik de Get-methode voor het ophalen van alle acties voor een schema.

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules/{Schedule ID}/actions?api-version=2015-03-20

Gebruik de Get-methode met de actie-ID voor het ophalen van een bepaalde actie voor een schema.

    armclient get /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/actions/{Action ID}?api-version=2015-03-20

### <a name="creating-or-editing-actions"></a>Maken of bewerken van acties
Gebruik de Put-methode met een actie-ID die uniek is voor de planning voor het maken van een nieuwe actie.  Wanneer u een actie in de OMS-console maakt, is een GUID voor het actie-ID.

> [!NOTE]
> De naam op voor alle opgeslagen zoekopdrachten, schema's en acties die zijn gemaakt met de Log Analytics-API moet in kleine letters.

Gebruik de Put-methode met een bestaande actie-ID voor de dezelfde opgeslagen zoekopdracht om te wijzigen die planning.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de planning.

De indeling van de aanvraag voor het maken van een nieuwe actie varieert per actietype zodat deze voorbeelden u in de volgende secties vindt.

### <a name="deleting-actions"></a>Acties worden verwijderd

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md). Acties zijn voor gebruikers die waarschuwingen naar Azure uitbreiden, nu beheerd met Azure Actiegroepen. Wanneer een werkruimte en de waarschuwingen worden uitgebreid naar Azure, kunt u ophalen of acties toevoegen met behulp van de [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).

Gebruik de Delete-methode met de actie-ID om te verwijderen van een actie.

    armclient delete /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/Actions/{Action ID}?api-version=2015-03-20

### <a name="alert-actions"></a>Waarschuwing acties
Een planning hebt slechts één meldingsactie.  Meldingen zijn een of meer van de secties in de volgende tabel.  Elk wordt nader hieronder beschreven.

| Sectie | Beschrijving | Gebruik |
|:--- |:--- |:--- |
| Drempelwaarde |Criteria voor wanneer de actie wordt uitgevoerd.| Vereist voor elke waarschuwing voordat of nadat ze zijn uitgebreid naar Azure. |
| Ernst |Het label dat wordt gebruikt om te classificeren waarschuwing wanneer deze worden geactiveerd.| Vereist voor elke waarschuwing voordat of nadat ze zijn uitgebreid naar Azure. |
| Actiegroepen |Id's van Azure ActionGroup waar acties vereist zijn opgegeven, zoals - e-mailberichten, SMSs, telefoongesprekken, Webhooks, Automation-Runbooks, ITSM Connectors, enzovoort.| Vereist wanneer waarschuwingen worden uitgebreid naar Azure|
| Acties aanpassen|De standaarduitvoer voor select acties van ActionGroup wijzigen| Optioneel voor elke waarschuwing kan worden gebruikt als waarschuwingen worden uitgebreid naar Azure. |
| EmailNotification |E-mail verzenden naar meerdere ontvangers. | Niet vereist, als waarschuwingen worden uitgebreid naar Azure|
| Herstel |Starten van een runbook in Azure Automation proberen om geïdentificeerde probleem te verhelpen. |Niet vereist, als waarschuwingen worden uitgebreid naar Azure|
| Webhookacties | Push-gegevens van waarschuwingen, naar de gewenste service als JSON |Niet vereist, als waarschuwingen worden uitgebreid naar Azure|

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md).

#### <a name="thresholds"></a>Drempelwaarden
Een waarschuwing actie mag slechts één drempelwaarde hebben.  Andere processen in deze actie worden uitgevoerd wanneer de resultaten van een opgeslagen zoekopdracht overeenkomen met de drempelwaarde in een actie die is gekoppeld aan die zoekopdracht.  Een actie kan alleen een drempelwaarde ook bevatten, zodat deze kan worden gebruikt met de acties die van andere typen die geen drempelwaarden bevatten.

De eigenschappen hebben drempelwaarden in de volgende tabel.

| Eigenschap | Beschrijving |
|:--- |:--- |
| Operator |Operator voor de drempelwaarde voor vergelijking. <br> gt = groter dan <br> lt = kleiner dan |
| Waarde |Waarde voor de drempelwaarde. |

Neem bijvoorbeeld een event-query met een Interval van 15 minuten, een TimeSpan-waarde van 30 minuten en een drempel van meer dan 10. In dit geval wordt de query wilt uitvoeren om de 15 minuten en een waarschuwing zou worden geactiveerd als het 10 gebeurtenissen die zijn gemaakt via een reeks van 30 minuten geretourneerd.

Hier volgt een voorbeeldantwoord voor een actie met alleen een drempelwaarde.  

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My threshold action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Version": 1
    }

De Put-methode met een unieke actie-ID gebruiken voor het maken van een nieuwe drempelwaarde-actie voor een schema.  

    $thresholdJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van een drempelwaarde voor een planning in te grijpen.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $thresholdJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson

#### <a name="severity"></a>Ernst
Log Analytics kunt u uw waarschuwingen classificeren in categorieën zodat u eenvoudiger beheer en selectie. De ernst van de waarschuwing die gedefinieerd is: informatief, waarschuwingen en kritieke. Deze worden toegewezen aan de schaal genormaliseerde ernst van waarschuwingen van Azure als:

|Log Analytics Ernstniveau  |Ernstniveau van waarschuwingen van Azure  |
|---------|---------|
|Kritieke |Sev 0|
|Waarschuwing |Sev 1|
|Informatief | Sev 2|

Hier volgt een voorbeeldantwoord voor een actie met alleen een drempelwaarde en ernst. 

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My threshold action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Severity": "critical",
        "Version": 1    }

De Put-methode met een unieke actie-ID gebruiken voor het maken van een nieuwe actie voor een schema met ernst.  

    $thresholdWithSevJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van een actie ernst voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $thresholdWithSevJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson

#### <a name="action-groups"></a>Actiegroepen
Alle waarschuwingen in Azure, gebruik actie groep als het standaardmechanisme voor het verwerken van acties. U kunt uw acties één keer opgeven en vervolgens de actiegroep meerdere waarschuwingen - koppelen in Azure met actie-groep. Zonder de noodzaak om te declareren dezelfde acties herhaaldelijk steeds opnieuw. Actie beheergroepen ondersteunen meerdere acties - zoals e-mail, SMS, telefoongesprek, ITSM verbinding, Automation-Runbook, Webhook URI en meer. 

Een planning hebt voor gebruikers die hun waarschuwingen hebt uitgebreid naar Azure - nu actie groepsgegevens doorgegeven samen met de drempelwaarde, te kunnen maken van een waarschuwing. E-details, Webhook-URL's, Runbook-automatisering details en andere acties moeten worden gedefinieerd aan de kant van een actie groep eerst maken voordat een waarschuwing; een kunt maken [groep acties van de Azure-Monitor](../monitoring-and-diagnostics/monitoring-action-groups.md) in de Portal of gebruik [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).

Wilt toevoegen koppeling van de actiegroep in een waarschuwing, geeft u de unieke ID van Azure Resource Manager van de actiegroep in de definitie van de waarschuwing. Hieronder vindt u een voorbeeld-afbeelding:

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ]
        },
        "Severity": "critical",
        "Version": 1
      },

Gebruik de Put-methode met een unieke actie-ID al een bestaande groep acties voor een planning koppelen.  Hier volgt een voorbeeld-afbeelding van het gebruik van.

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van de groep van een actie die is gekoppeld voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

#### <a name="customize-actions"></a>Acties aanpassen
Door standaardacties, volgt u de standaardsjabloon en de indeling voor meldingen. Maar gebruiker kunt aanpassen sommige acties, zelfs als ze worden beheerd door de Actiegroepen. Op dit moment is aanpassing mogelijk voor het onderwerp van E-mail en Webhook-nettolading.

##### <a name="customize-e-mail-subject-for-action-group"></a>E-mailonderwerp voor actie groep aanpassen
Het onderwerp e-mail voor waarschuwingen is standaard: melding van waarschuwingen <AlertName> voor <WorkspaceName>. Maar deze kan worden aangepast, zodat u kunt specifieke woorden of labels - kunt u eenvoudig alvast filterregels in je postvak in. De details van aanpassen e-header moeten verzenden samen met ActionGroup details, zoals in het voorbeeld hieronder.

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ]
          "CustomEmailSubject": "Azure Alert fired"
        },
        "Severity": "critical",
        "Version": 1
      },

Gebruik de Put-methode met een unieke actie-ID al een bestaande groep voor acties koppelen aan de aanpassing voor een schema.  Hier volgt een voorbeeld-afbeelding van het gebruik van.

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van de groep van een actie die is gekoppeld voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired' }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

##### <a name="customize-webhook-payload-for-action-group"></a>De nettolading van de Webhook aanpassen voor actie-groep
De webhook verzonden via de actie-groep voor logboekanalyse heeft standaard een vaste structuur. Maar een JSON-nettolading kunt aanpassen met behulp van specifieke variabelen die worden ondersteund, om te voldoen aan de vereisten van de webhook-eindpunt. Zie voor meer informatie [Webhook-actie voor waarschuwingsregels logboek](../monitoring-and-diagnostics/monitor-alerts-unified-log-webhook.md). 

De details van de webhook aanpassen moeten samen met details ActionGroup verzenden en zal worden toegepast op alle Webhook URI opgegeven in de groep in te grijpen. zoals in het voorbeeld hieronder.

     "etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
      "properties": {
        "Type": "Alert",
        "Name": "test-alert",
        "Description": "I need to put a descriptio here",
        "Threshold": {
          "Operator": "gt",
          "Value": 12
        },
        "AzNsNotification": {
          "GroupIds": [
            "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
          ]
          "CustomWebhookPayload": "{\"field1\":\"value1\",\"field2\":\"value2\"}",
          "CustomEmailSubject": "Azure Alert fired"
        },
        "Severity": "critical",
        "Version": 1
      },

Gebruik de Put-methode met een unieke actie-ID al een bestaande groep voor acties koppelen aan de aanpassing voor een schema.  Hier volgt een voorbeeld-afbeelding van het gebruik van.

    $AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van de groep van een actie die is gekoppeld voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}' }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson

#### <a name="email-notification"></a>E-mailmelding
E-mailmeldingen een e-mail verzenden naar een of meer geadresseerden.  Ze bevatten de eigenschappen in de volgende tabel.

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md). Voor gebruikers die waarschuwingen naar Azure uitbreiden, worden de acties zoals e-mailmelding nu beheerd in Azure Actiegroepen. Wanneer een werkruimte en de waarschuwingen worden uitgebreid naar Azure, kunt u ophalen of acties toevoegen met behulp van de [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).
   

| Eigenschap | Beschrijving |
|:--- |:--- |
| Geadresseerden |Lijst met e-mailadressen. |
| Onderwerp |Het onderwerp van het e-mailbericht. |
| Bijlage |Bijlagen worden momenteel niet ondersteund, zodat dit altijd de waarde 'None'. |

Hier volgt een voorbeeldantwoord voor een actie van de melding e-mailbericht met een drempelwaarde.  

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My email action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "EmailNotification": {
            "Recipients": [
                "recipient1@contoso.com",
                "recipient2@contoso.com"
            ],
            "Subject": "This is the subject",
            "Attachment": "None"
        },
        "Version": 1
    }

De Put-methode met een unieke actie-ID gebruiken voor het maken van een nieuwe e-actie voor een schema.  Het volgende voorbeeld wordt een e-mailbericht met een drempelwaarde, zodat de e-mail wordt verzonden wanneer de drempelwaarde, de resultaten van de opgeslagen zoekopdracht overschrijden.

    $emailJson = "{'properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myemailaction?api-version=2015-03-20 $emailJson

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van een e-actie voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $emailJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myemailaction?api-version=2015-03-20 $emailJson

#### <a name="remediation-actions"></a>Herstelacties
Herstelbewerkingen starten van een runbook in Azure Automation waarmee wordt geprobeerd het probleem dat wordt geïdentificeerd door de waarschuwing te verhelpen.  U moet een webhook voor het runbook dat is gebruikt in een herstelactie maken en geeft u de URI in de eigenschap WebhookUri.  Wanneer u deze actie met de OMS-console maakt, wordt automatisch een nieuwe webhook gemaakt voor het runbook.

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md). Voor gebruikers die waarschuwingen naar Azure uitbreiden, worden acties zoals herstel met behulp van runbook nu beheerd met Azure Actiegroepen. Wanneer een werkruimte en de waarschuwingen worden uitgebreid naar Azure, kunt u ophalen of acties toevoegen met behulp van de [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).

Herstelbewerkingen bevatten de eigenschappen in de volgende tabel.

| Eigenschap | Beschrijving |
|:--- |:--- |
| RunbookName |De naam van het runbook. Dit moet overeenkomen met een gepubliceerd runbook in de automation-account geconfigureerd in de Automation-oplossing in de OMS-werkruimte. |
| WebhookUri |De URI van de webhook. |
| Vervaldatum |De vervaldatum en -tijd van de webhook.  Als de webhook niet beschikt over een verlopen, kan dit een geldige datum in de toekomst zijn. |

Hier volgt een voorbeeldantwoord voor een herstelactie met een drempelwaarde.

    "etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
    "properties": {
        "Type": "Alert",
        "Name": "My remediation action",
        "Threshold": {
            "Operator": "gt",
            "Value": 10
        },
        "Remediation": {
            "RunbookName": "My-Runbook",
            "WebhookUri": "https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d",
            "Expiry": "2018-02-25T18:27:20"
            },
        "Version": 1
    }

De Put-methode met een unieke actie-ID gebruiken voor het maken van een nieuwe herstelactie voor een schema.  Het volgende voorbeeld wordt een herstelbewerking met een drempelwaarde, zodat het runbook wordt gestart wanneer de resultaten van de opgeslagen zoekopdracht de drempelwaarde overschrijden.

    $remediateJson = "{'properties': { 'Type':'Alert', 'Name': 'My Remediation Action', 'Version':'1', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'Remediation': {'RunbookName': 'My-Runbook', 'WebhookUri':'https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d', 'Expiry':'2018-02-25T18:27:20Z'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myremediationaction?api-version=2015-03-20 $remediateJson

Gebruik de Put-methode met een bestaande actie-ID een herstelactie voor een schema wijzigen.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $remediateJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Type':'Alert', 'Name': 'My Remediation Action', 'Version':'1', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'Remediation': {'RunbookName': 'My-Runbook', 'WebhookUri':'https://s1events.azure-automation.net/webhooks?token=4jCibOjO3w4W2Cfg%2b2NkjLYdafnusaG6i8tnP8h%2fNNg%3d', 'Expiry':'2018-02-25T18:27:20Z'} }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myremediationaction?api-version=2015-03-20 $remediateJson

#### <a name="example"></a>Voorbeeld
Hier volgt een voorbeeld van een volledige een nieuw e-waarschuwing wilt maken.  Hiermee maakt u een nieuw schema samen met een actie met een drempelwaarde en e-mailbericht.

    $subscriptionId = "3d56705e-5b26-5bcc-9368-dbc8d2fafbfc"
    $resourceGroup  = "MyResourceGroup"    
    $workspaceName    = "MyWorkspace"
    $searchId       = "MySearch"
    $scheduleId     = "MySchedule"
    $thresholdId    = "MyThreshold"
    $actionId       = "MyEmailAction"

    $scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Active':'true' }"
    armclient put /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/savedSearches/$searchId/schedules/$scheduleId/?api-version=2015-03-20 $scheduleJson

    $emailJson = "{'properties': { 'Name': 'MyEmailAction', 'Version':'1', 'Severity':'Warning', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 }, 'EmailNotification': {'Recipients': ['recipient1@contoso.com', 'recipient2@contoso.com'], 'Subject':'This is the subject', 'Attachment':'None'} }"
    armclient put /subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.OperationalInsights/workspaces/$workspaceName/savedSearches/$searchId/schedules/$scheduleId/actions/$actionId/?api-version=2015-03-20 $emailJson

#### <a name="webhook-actions"></a>Webhookacties
Een proces starten webhookacties door het aanroepen van een URL en het eventueel geven een nettolading worden verzonden.  Ze zijn vergelijkbaar met herstelacties maar ze zijn bedoeld voor webhooks die van processen dan Azure Automation-runbooks gebruikmaken mogelijk.  Ze bieden ook de aanvullende mogelijkheid bieden een nettolading moet worden geleverd aan het externe proces.

> [!NOTE]
> 14 mei 2018, vanaf wordt alle waarschuwingen in een werkruimte automatisch uitgebreid naar Azure. Een gebruiker kunt uitbreiden waarschuwingen naar Azure vrijwillig voordat 14 mei 2018 initiëren. Zie voor meer informatie [waarschuwingen uitbreiden naar Azure van OMS](../monitoring-and-diagnostics/monitoring-alerts-extend.md). Voor gebruikers die waarschuwingen naar Azure uitbreiden, worden de acties zoals Webhook nu beheerd in Azure Actiegroepen. Wanneer een werkruimte en de waarschuwingen worden uitgebreid naar Azure, kunt u ophalen of acties toevoegen met behulp van de [actie groep API](https://docs.microsoft.com/en-us/rest/api/monitor/actiongroups).


Webhookacties hoeven niet een drempelwaarde, maar in plaats daarvan moeten worden toegevoegd aan een schema dat een waarschuwing actie met een drempelwaarde is.  

Hier volgt een voorbeeldantwoord voor webhook actie en een bijbehorende waarschuwingen met een drempelwaarde.

    {
        "__metadata": {},
        "value": [
            {
                "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/bwren/savedSearches/2d1b30fb-7f48-4de5-9614-79ee244b52de/schedules/b80f5621-7217-4007-b32d-165d14377093/Actions/72884702-acf9-4653-bb67-f42436b342b4",
                "etag": "W/\"datetime'2016-02-26T20%3A25%3A00.6862124Z'\"",
                "properties": {
                    "Type": "Webhook",
                    "Name": "My Webhook Action",
                    "WebhookUri": "https://oaaswebhookdf.cloudapp.net/webhooks?token=VfkYTIlpk%2fc%2bJBP",
                    "CustomPayload": "{\"fielld1\":\"value1\",\"field2\":\"value2\"}",
                    "Version": 1
                }
            },
            {
                "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/bwren/savedSearches/2d1b30fb-7f48-4de5-9614-79ee244b52de/schedules/b80f5621-7217-4007-b32d-165d14377093/Actions/90a27cf8-71b7-4df2-b04f-54ed01f1e4b6",
                "etag": "W/\"datetime'2016-02-26T20%3A25%3A00.565204Z'\"",
                "properties": {
                    "Type": "Alert",
                    "Name": "Threshold for my webhook action",
                    "Threshold": {
                        "Operator": "gt",
                        "Value": 10
                    },
                    "Version": 1
                }
            }
        ]
    }

##### <a name="create-or-edit-a-webhook-action"></a>Maken of bewerken van een webhook-actie
De Put-methode met een unieke actie-ID gebruiken voor het maken van een nieuwe webhook-actie voor een schema.  Het volgende voorbeeld maakt een actie Webhook en een waarschuwing met een drempelwaarde zodat de webhook wordt geactiveerd wanneer de resultaten van de opgeslagen zoekopdracht de drempelwaarde overschrijden.

    $thresholdAction = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdAction

    $webhookAction = "{'properties': {'Type': 'Webhook', 'Name': 'My Webhook", 'WebhookUri': 'https://oaaswebhookdf.cloudapp.net/webhooks?token=VrkYTKlhk%2fc%2bKBP', 'CustomPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}', 'Version': 1 }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mywebhookaction?api-version=2015-03-20 $webhookAction

Gebruik de Put-methode met een bestaande actie-ID te wijzigen van een webhook-actie voor een schema.  De hoofdtekst van de aanvraag vergezeld gaan van de etag van de actie.

    $webhookAction = "{'etag': 'W/\"datetime'2016-02-26T20%3A25%3A00.6862124Z'\"','properties': {'Type': 'Webhook', 'Name': 'My Webhook", 'WebhookUri': 'https://oaaswebhookdf.cloudapp.net/webhooks?token=VrkYTKlhk%2fc%2bKBP', 'CustomPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}', 'Version': 1 }"
    armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mywebhookaction?api-version=2015-03-20 $webhookAction


## <a name="next-steps"></a>Volgende stappen
* Gebruik de [REST-API om logboek zoekopdrachten](log-analytics-log-search-api.md) in logboekanalyse.
* Meer informatie over [waarschuwingen melden waarschuwingen van azure](../monitoring-and-diagnostics/monitor-alerts-unified-log.md)

