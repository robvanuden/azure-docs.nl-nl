---
title: Verschillen en overwegingen voor virtuele machines in Azure Stack | Microsoft Docs
description: Meer informatie over de verschillen en overwegingen bij het werken met virtuele machines in Azure-Stack.
services: azure-stack
documentationcenter: 
author: brenduns
manager: femila
editor: 
ms.assetid: 6613946D-114C-441A-9F74-38E35DF0A7D7
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/23/2018
ms.author: brenduns
ms.openlocfilehash: 50c0f293ac669ade4e45a5f45b0adf9a7c4b6c36
ms.sourcegitcommit: a36a1ae91968de3fd68ff2f0c1697effbb210ba8
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/17/2018
---
# <a name="considerations-for-virtual-machines-in-azure-stack"></a>Overwegingen voor virtuele Machines in Azure Stack

*Van toepassing op: Azure Stack geïntegreerde systemen en Azure Stack Development Kit*

Virtuele machines zijn op aanvraag, schaalbare computerbronnen die worden aangeboden door de Azure-Stack. Wanneer u virtuele Machines gebruikt, moet u begrijpen dat er verschillen zijn tussen de functies die beschikbaar in Azure zijn en Azure-Stack. Dit artikel bevat een overzicht van de unieke overwegingen voor virtuele Machines en de bijbehorende functies in Azure-Stack. Zie voor meer informatie over belangrijke verschillen tussen Azure-Stack en Azure, de [belangrijke overwegingen](azure-stack-considerations.md) artikel.

## <a name="cheat-sheet-virtual-machine-differences"></a>Blad cheats: verschillen van de virtuele machine

| Functie | Azure (global) | Azure Stack |
| --- | --- | --- |
| Installatiekopieën van virtuele machines | Azure Marketplace bevat afbeeldingen die u gebruiken kunt voor het maken van een virtuele machine. Zie de [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/compute?subcategories=virtual-machine-images&page=1) pagina om de lijst met afbeeldingen die beschikbaar in Azure Marketplace zijn weer te geven. | Standaard bevat niet alle installatiekopieën beschikbaar zijn in de Stack van Azure marketplace. De beheerder van de Azure-Stack cloud moet publiceren of installatiekopieën downloaden naar de Stack van Azure marketplace voordat gebruikers ze kunnen gebruiken. |
| Grootten van virtuele machines | Azure ondersteunt een groot aantal verschillende grootten voor virtuele machines. Raadpleeg voor meer informatie over de beschikbare grootten en opties voor de [Windows virtuele machines grootten](../../virtual-machines/virtual-machines-windows-sizes.md) en [grootten van virtuele machines van Linux](../../virtual-machines/linux/sizes.md) onderwerpen. | Azure-Stack ondersteunt een subset van de grootte van virtuele machines die beschikbaar in Azure zijn. Als u wilt weergeven in de lijst met ondersteunde grootten, verwijzen naar de [grootten van virtuele machines](#virtual-machine-sizes) sectie van dit artikel. |
| Quota's voor virtuele machine | [De quotalimieten voor](../../azure-subscription-service-limits.md#service-specific-limits) zijn ingesteld door Microsoft | De beheerder van de Azure-Stack cloud moet quota toewijzen voordat ze aan virtuele machines te aan hun gebruikers bieden. |
| Extensies van de virtuele machine |Azure biedt ondersteuning voor tal van uitbreidingen van de virtuele machine. Raadpleeg voor meer informatie over de beschikbare uitbreidingen, de [uitbreidingen van de virtuele machine en functies](../../virtual-machines/windows/extensions-features.md) artikel.| Azure-Stack ondersteunt een subset van de uitbreidingen die beschikbaar in Azure zijn en elk van de extensie specifieke versies hebben. De beheerder van de Azure-Stack cloud kunt kiezen welke uitbreidingen die beschikbaar voor gebruikers. Raadpleeg de lijst met ondersteunde extensies wilt weergeven, de [uitbreidingen van de virtuele machine](#virtual-machine-extensions) sectie van dit artikel. |
| Netwerk voor virtuele machines | Openbare IP-adressen toegewezen aan virtuele machines van tenants zijn via Internet toegankelijk.<br><br><br>Virtuele Machines in Azure heeft een vaste DNS-naam | Openbare IP-adressen toegewezen aan de virtuele machine van een tenant zijn binnen de omgeving van Azure Stack Development Kit alleen toegankelijk. Een gebruiker moet toegang hebben tot de Azure-Stack Development Kit via [RDP](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop) of [VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) verbinding maken met een virtuele machine die wordt gemaakt in Azure-Stack.<br><br>Virtuele machines die zijn gemaakt binnen een specifiek exemplaar van de Azure-Stack is een DNS-naam op basis van de waarde die is geconfigureerd door de beheerder van de cloud. |
| Virtuele-machineopslag | Ondersteunt [schijven die worden beheerd.](../../virtual-machines/windows/managed-disks-overview.md) | Beheerde schijven zijn nog niet ondersteund in Azure-Stack. |
| API-versies | Azure heeft altijd de nieuwste API-versies voor alle functies van de virtuele machine. | Azure ondersteunt Stack specifieke Azure-services en de specifieke API-versies voor deze services. Raadpleeg de lijst met ondersteunde API-versies wilt weergeven, de [API-versies](#api-versions) sectie van dit artikel. |
|Virtuele machine beschikbaarheidssets|Meerdere domeinen met fouten (2 of 3 per regio)<br>Meerdere domeinen van de update<br>Ondersteuning voor schijven beheerd|Meerdere domeinen met fouten (2 of 3 per regio)<br>Meerdere domeinen van de update (maximaal 20)<br>Er is geen ondersteuning voor beheerde schijven|
|Virtuele-machineschaalsets|Automatisch geschaald ondersteund|Automatisch geschaald niet ondersteund.<br>Meer exemplaren toevoegen aan een schaal ingesteld met de portal, Resource Manager-sjablonen of PowerShell.

## <a name="virtual-machine-sizes"></a>Grootten van virtuele machines
Azure legt limieten op verschillende manieren om te voorkomen dat overmatig van resources (lokale en serviceniveau-server). De ervaring voor de tenant kunt celinhoud sommige limieten van een verbruik tenants van resource afnemen wanneer een ruis neighbor overconsumes resources. 
- Voor netwerken uitgaande van de virtuele machine zijn er bandbreedte caps aanwezig. Hoofdletters in Azure-Stack overeen met de caps in Azure.  
- Voor opslagbronnen implementeert Azure Stack IOPs opslaglimieten om te voorkomen dat basic overmatig van bronnen door tenants voor toegang tot opslag. 
- Voor virtuele machines met meerdere schijven van de bijgesloten gegevens is de maximale doorvoer van elke afzonderlijke gegevensschijf 500 IOPS aan voor HHDs en 2300 IOPS voor SSD's.

De volgende tabel bevat de virtuele machines die worden ondersteund op Azure-Stack samen met hun configuratie:

| Type           | Grootte          | Aantal ondersteunde formaten |
| ---------------| ------------- | ------------------------ |
|Algemeen doel |Basic A        |[A0 - A4](azure-stack-vm-sizes.md#basic-a)                   |
|Algemeen doel |Standard A     |[A0 - A7](azure-stack-vm-sizes.md#standard-a)              |
|Algemeen doel |D-serie       |[D1 - D4](azure-stack-vm-sizes.md#d-series)              |
|Algemeen doel |Dv2-serie     |[D1_v2 - D5_v2](azure-stack-vm-sizes.md#ds-series)        |
|Algemeen doel |DS-serie      |[DS1 - DS4](azure-stack-vm-sizes.md#dv2-series)            |
|Algemeen doel |DSv2-serie    |[DS1_v2 - DS5_v2](azure-stack-vm-sizes.md#dsv2-series)      |
|Geoptimaliseerd geheugen|D-serie       |[D11 - D14](azure-stack-vm-sizes.md#mo-d)            |
|Geoptimaliseerd geheugen|DS-serie      |[DS11 - DS14](azure-stack-vm-sizes.md#mo-ds)|
|Geoptimaliseerd geheugen|Dv2-serie     |[D11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dv2)     |
|Geoptimaliseerd geheugen|DSv2-series -  |[DS11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dsv2)    |

Grootte van virtuele machines en hun bijbehorende resource hoeveelheden zijn consistent met de Azure-Stack en Azure. Deze consistentie bevat bijvoorbeeld de hoeveelheid geheugen, het aantal kernen en nummer of grootte van gegevensschijven dat kan worden gemaakt. Prestaties van de hetzelfde VM-grootte in Azure-Stack is afhankelijk van de onderliggende kenmerken van een bepaalde Azure Stack-omgeving.

## <a name="virtual-machine-extensions"></a>Extensies van de virtuele machine

 De Azure-Stack bevat een kleine set van extensies. Updates en aanvullende extensies en zijn beschikbaar via de Marketplace-syndicatiefeed.

Gebruik de volgende PowerShell-script om de lijst met extensies van virtuele machine die beschikbaar in uw Azure-Stack-omgeving zijn:

```powershell
Get-AzureRmVmImagePublisher -Location local | `
  Get-AzureRmVMExtensionImageType | `
  Get-AzureRmVMExtensionImage | `
  Select Type, Version | `
  Format-Table -Property * -AutoSize
```

## <a name="api-versions"></a>API-versies

Functies van de virtuele machine in Azure-Stack ondersteunen de volgende API-versies:

![Brontypen VM](media/azure-stack-vm-considerations/vm-resoource-types.png)

Ophalen van de API-versies voor de functies van de virtuele machine die beschikbaar in uw Azure-Stack-omgeving zijn kunt u het volgende PowerShell-script:

```powershell
Get-AzureRmResourceProvider | `
  Select ProviderNamespace -Expand ResourceTypes | `
  Select * -Expand ApiVersions | `
  Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} | `
  where-Object {$_.ProviderNamespace -like “Microsoft.compute”}
```
De lijst met ondersteunde resourcetypen en API-versies kan verschillen als de operator cloud uw Azure-Stack-omgeving naar een nieuwere versie bijwerkt.

## <a name="next-steps"></a>Volgende stappen

[Een virtuele Windows-machine maken met PowerShell in Azure-Stack](azure-stack-quick-create-vm-windows-powershell.md)
