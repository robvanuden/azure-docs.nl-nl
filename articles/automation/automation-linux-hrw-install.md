---
title: Hybrid Runbook Worker voor Azure Automation in Linux
description: In dit artikel bevat informatie over het installeren van een Azure Automation Hybrid Runbook Worker die u kunt het uitvoeren van runbooks op Linux-computers in uw lokale datacentrum of cloudomgeving.
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: article
manager: carmonm
ms.openlocfilehash: b4559afa9294111eaa1f20fdf295d1fb26dcc994
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/28/2018
---
# <a name="how-to-deploy-a-linux-hybrid-runbook-worker"></a>Het implementeren van een Linux hybride Runbook Worker

Runbooks in Azure Automation heeft geen toegang tot resources in andere clouds of in uw on-premises omgeving omdat ze worden uitgevoerd in de Azure-cloud. De Hybrid Runbook Worker-functie van Azure Automation kunt u direct op de computer die als host fungeert voor de rol en op basis van bronnen in de omgeving voor het beheren van de lokale bronnen runbooks worden uitgevoerd. Runbooks zijn opgeslagen en beheerd in Azure Automation en vervolgens aan een of meer specifieke computers geleverd.

Deze functionaliteit wordt geïllustreerd in de volgende afbeelding:<br>  

![Overzicht van hybride Runbook Worker](media/automation-offering-get-started/automation-infradiagram-networkcomms.png)

Zie voor een technisch overzicht van de Hybrid Runbook Worker-rol [Automation architectuuroverzicht](automation-offering-get-started.md#automation-architecture-overview). Controleer de volgende informatie met betrekking tot de [hardware- en softwarevereisten](automation-offering-get-started.md#hybrid-runbook-worker) en [informatie voor het voorbereiden van uw netwerk](automation-offering-get-started.md#network-planning) voordat u begint met het implementeren van een hybride Runbook Worker. Nadat u hebt een runbook worker is geïmplementeerd, controleren [runbooks worden uitgevoerd op een hybride Runbook Worker](automation-hrw-run-runbooks.md) voor informatie over het configureren van uw runbooks voor het automatiseren van processen in uw on-premises datacentrum of andere cloudomgeving.     

## <a name="hybrid-runbook-worker-groups"></a>Hybrid Runbook Worker-groepen
Elke Hybrid Runbook Worker is lid van een hybride Runbook Worker-groep die u opgeeft wanneer u de agent installeert. Een groep kan één agent bevatten, maar u kunt meerdere agents installeren in een groep voor hoge beschikbaarheid.

Wanneer u een runbook op een hybride Runbook Worker start, geeft u de groep die op wordt uitgevoerd. De leden van de groep bepalen welke worker-services de aanvraag. U kunt een bepaalde worker niet opgeven.

## <a name="installing-linux-hybrid-runbook-worker"></a>Linux hybride Runbook Worker installeren
Als u wilt installeren en configureren van een hybride Runbook Worker op uw Linux-computer, moet u een rechte doorsturen-proces voor het handmatig installeren en configureren van de rol volgen. Het inschakelen van vereist de **Automation Hybrid Worker** oplossing in de werkruimte voor logboekanalyse en voert u een reeks opdrachten voor het registreren van de computer als een werknemer en toe te voegen aan een nieuwe of bestaande groep. 

Voordat u doorgaat, moet u de werkruimte voor logboekanalyse die uw Automation-account is gekoppeld aan, en ook de primaire sleutel voor uw Automation-account. U kunt zowel vanuit de portal vinden door uw Automation-account selecteren en het selecteren van **werkruimte** voor de werkruimte-ID en het selecteren van **sleutels** voor de primaire sleutel.  

1.  Schakel de oplossing 'Automation Hybrid Worker' in Azure. Dit kan worden gedaan door:

   1. Toevoegen de **Automation Hybrid Worker** oplossing aan uw abonnement met de procedure op [beheeroplossingen voor logboekanalyse toevoegen aan uw werkruimte](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-add-solutions).
   2. Voer de volgende cmdlet uit:

        ```powershell
         $null = Set-AzureRmOperationalInsightsIntelligencePack -ResourceGroupName  <ResourceGroupName> -WorkspaceName <WorkspaceName> -IntelligencePackName  "AzureAutomation" -Enabled $true
        ```

2.  Voer de volgende opdracht, het wijzigen van de waarden voor parameters *-w*, *-k*, *-g*, en *-e*. Voor de *-g* parameter Vervang de waarde met de naam van de Hybrid Runbook Worker-groep die moet worden toegevoegd aan de nieuwe Linux hybride Runbook Worker. Als de naam niet al in uw Automation-account bestaat, wordt een nieuwe hybride Runbook Worker-groep bestaat met die naam.
    
    ```python
    sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <LogAnalyticsworkspaceId> -k <AutomationSharedKey> -g <hybridgroupname> -e <automationendpoint>
    ```
3. Nadat de opdracht voltooid is, ziet de blade Hybrid Worker-groepen in de Azure portal de nieuwe groep en het aantal leden of als een bestaande groep, het aantal leden wordt verhoogd. U kunt de groep in de lijst selecteren op de **Hybrid Worker-groepen** blade en selecteer de **Hybrid Workers** tegel. Op de **Hybrid Workers** blade ziet u elk lid van de groep die wordt vermeld.  


## <a name="turning-off-signature-validation"></a>Het uitschakelen van validatie van handtekening 
Standaard is Linux Hybrid Runbook Workers handtekeningvalidatie nodig. Als u een niet-ondertekende runbook op basis van een werknemer uitvoeren, ziet u een fout met "Handtekeningvalidatie is mislukt". Als wilt uitschakelen handtekeningvalidatie, voer de volgende opdracht, de tweede parameter vervangen door uw Log Analytics-werkruimte-ID:

 ```python
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <LogAnalyticsworkspaceId>
 ```

## <a name="supported-runbook-types"></a>Ondersteunde runbooktypen

Linux Hybrid Runbook Workers bieden geen ondersteuning voor de volledige set van runbooktypen die zijn gevonden binnen Azure automation.

De volgende runbooktypen werken op een Linux hybride Worker:

* Python 2
* PowerShell
 
De volgende runbooktypen werken niet op een Linux hybride Worker:

* PowerShell-werkstroom
* Grafische
* Grafische PowerShell Workflow

## <a name="next-steps"></a>Volgende stappen

* Bekijk [runbooks worden uitgevoerd op een hybride Runbook Worker](automation-hrw-run-runbooks.md) voor informatie over het configureren van uw runbooks voor het automatiseren van processen in uw on-premises datacentrum of andere cloudomgeving.
* Zie voor instructies over het verwijderen van Hybrid Runbook Workers [verwijderen Azure Automation Hybrid Runbook Workers](automation-remove-hrw.md)
