---
title: bestand opnemen
description: bestand opnemen
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/03/2018
ms.author: cynthn;kareni
ms.custom: include file
ms.openlocfilehash: 81357bce92bb8bd2f77f7aaabc8e3b1d49047a1b
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/05/2018
---
**Laatste document update**: 3 April 3:00 PM PST.

De recente openbaarmaking van een [nieuwe klasse van CPU-beveiligingsproblemen](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002) die bekend staat als speculatieve uitvoering side-kanaal aanvallen heeft geleid tot vragen van klanten meer duidelijkheid te zoeken.  

Microsoft heeft oplossingen geïmplementeerd op onze cloudservices. De infrastructuur die wordt uitgevoerd van Azure en werkbelastingen van de klant van elkaar isoleert is beveiligd.  Dit betekent dat andere klanten uitgevoerd op Azure uw toepassing met behulp van deze beveiligingslekken kunnen geen aanvallen.

Bovendien Azure breidt het gebruik van [geheugen behouden onderhoud](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates#memory-preserving-maintenance) indien mogelijk onderbreken van de virtuele machine voor maximaal 30 seconden, terwijl de host is bijgewerkt of de virtuele machine wordt verplaatst naar een host al bijgewerkt.  Geheugen behouden onderhoud verdere klant gevolgen minimaliseert en hoeft u opnieuw wordt opgestart.  Azure maakt gebruik van deze methoden wanneer het hele systeem updates indienen bij de host.

> [!NOTE] 
> In latere februari 2018, Intel Corporation gepubliceerd bijgewerkte [Microcode revisie richtlijnen](https://newsroom.intel.com/wp-content/uploads/sites/11/2018/03/microcode-update-guidance.pdf) op de status van hun microcode-versies die de stabiliteit te verbeteren en beperken dat recente beveiligingsproblemen vermeld door [Google Project nul](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html). De oplossingen die door Azure op [3 januari 2018](https://azure.microsoft.com/en-us/blog/securing-azure-customers-from-cpu-vulnerability/) worden niet beïnvloed door van Intel microcode update. Microsoft al ingevoerd sterk oplossingen Azure om klanten te beschermen van andere virtuele machines in Azure.  
>
> Van Intel microcode adressen variant 2 Spectre ([CVE-2017-5715](https://www.cve.mitre.org/cgi-bin/cvename.cgi?name=2017-5715) of vertakking doel injectie) te beschermen tegen aanvallen die alleen zijn van toepassing waarin u gedeelde of niet-vertrouwde werkbelasting uitvoeren binnen uw virtuele machines in Azure. Onze technici test de stabiliteit om te beperken van invloed op de prestaties van de microcode, voordat het beschikbaar maken op Azure-klanten.  Als klanten heel weinig niet-vertrouwde werkbelastingen binnen hun virtuele machines worden uitgevoerd, moet de meeste klanten niet inschakelen van deze mogelijkheid eenmaal uitgebracht. 
>
> Deze pagina wordt bijgewerkt als u meer informatie beschikbaar is.  






## <a name="keeping-your-operating-systems-up-to-date"></a>Uw besturingssystemen up-to-date te houden

Tijdens een update van het besturingssysteem niet vereist is voor het isoleren van uw toepassingen die worden uitgevoerd op Azure van andere klanten uitgevoerd op Azure, maar het is altijd een best practice uw versies van het besturingssysteem up-to-date kunt houden. De januari 2018 en hoger [beveiliging updatepakketten voor Windows](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002) bevatten oplossingen voor deze beveiligingslekken.

In de volgende aanbiedingen vindt hier u de aanbevolen acties in uw besturingssysteem bijwerken: 

<table>
<tr>
<th>Aanbieding</th> <th>Aanbevolen actie </th>
</tr>
<tr>
<td>Azure Cloud Services </td>  <td>Automatisch bijwerken inschakelen of zorg ervoor dat u de nieuwste Gast-besturingssysteem op worden uitgevoerd.</td>
</tr>
<tr>
<td>Azure Linux virtuele Machines</td> <td>Updates installeren vanaf uw besturingssysteem provider indien beschikbaar. </td>
</tr>
<tr>
<td>Windows Azure virtuele Machines </td> <td>Controleer of u een ondersteunde antivirusprogramma toepassing worden uitgevoerd voordat u updates voor het besturingssysteem installeert. Neem contact op met de leverancier van uw antivirussoftware voor informatie over de compatibiliteit.<p> Installeer de [totalisering van de beveiliging januari](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002). </p></td>
</tr>
<tr>
<td>Andere Azure PaaS-Services</td> <td>Er is geen actie nodig is voor klanten die gebruikmaken van deze services. Azure blijft automatisch de versies van het besturingssysteem up-to-date. </td>
</tr>
</table>

## <a name="additional-guidance-if-you-are-running-untrusted-code"></a>Aanvullende richtlijnen als u werkt met niet-vertrouwde code 

Geen aanvullende klant-actie is vereist tenzij u niet-vertrouwde code uitvoert. Als u de code die u niet vertrouwt toestaat (u kunt bijvoorbeeld een van uw klanten voor het uploaden van een binair of codefragment die u vervolgens wordt uitgevoerd in de cloud in uw toepassing), en vervolgens de volgende aanvullende stappen moeten worden genomen.  


### <a name="windows"></a>Windows 
Als u met behulp van Windows en niet-vertrouwde code te hosten, moet u ook een aangeroepen Kernel virtueel adres (KVA) schaduwkopie maken die zorgt voor extra beveiliging tegen speculatieve beveiligingslekken side-kanaal (specifiek voor Windows-functie inschakelen VARIANT 3 smelten, [CVE-2017-5754](https://www.cve.mitre.org/cgi-bin/cvename.cgi?name=2017-5754), of het laden van de cache rogue gegevens). Deze functie is standaard uitgeschakeld en kan invloed hebben op prestaties als ingeschakeld. Ga als volgt [Windows Server KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) instructies voor het inschakelen van beveiliging op de Server. Als u Azure Cloud Services uitvoert, controleert u of dat u altijd WA-GUEST-OS-5.15_201801-01 of WA-GUEST-OS-4.50_201801-01 (beschikbaar vanaf op 10 januari 2018) en schakelt u het register sleutel via een taak starten.


### <a name="linux"></a>Linux
Als u met behulp van Linux en niet-vertrouwde code te hosten, moet u ook Linux bijwerken naar een recentere versie waarop kernel tabel pagina isolatie (KPTI) die worden gescheiden van de pagina tabellen die worden gebruikt door de kernel van de referenties die horen bij de gebruikersruimte die is geïmplementeerd. Deze oplossingen vereisen een update voor de Linux-besturingssysteem en kunnen worden verkregen van uw provider distributie indien beschikbaar. Uw OS-provider kunt u zien of beveiliging zijn ingeschakeld of standaard uitgeschakeld.



## <a name="next-steps"></a>Volgende stappen

Zie voor meer informatie, [beveiligen van Azure-klanten CPU beveiligingslek](https://azure.microsoft.com/blog/securing-azure-customers-from-cpu-vulnerability/).
