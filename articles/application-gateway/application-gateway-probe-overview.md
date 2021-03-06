---
title: Overzicht van statusbewaking voor Azure Application Gateway
description: Meer informatie over de bewakingsmogelijkheden in Azure Application Gateway
services: application-gateway
documentationcenter: na
author: vhorne
manager: jpconnock
tags: azure-resource-manager
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/30/2018
ms.author: victorh
ms.openlocfilehash: 2f62f01c1178f9529eb46051f088affccc5279a7
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/03/2018
---
# <a name="application-gateway-health-monitoring-overview"></a>Overzicht van Application Gateway health monitoring

Azure Application Gateway standaard bewaakt de status van alle resources in de back-end-pool en worden alle bronnen die niet in orde beschouwd uit de groep automatisch verwijderd. Toepassingsgateway blijft de slechte instanties bewaken en terug naar de goede back-end-pool toegevoegd zodra ze beschikbaar en op statuscontroles reageren. Toepassingsgateway verzendt dat de statuscontroles met dezelfde poort die is gedefinieerd in de back-end-HTTP-instellingen. Deze configuratie zorgt ervoor dat de test is het testen van dezelfde poort die klanten gebruik maken van zouden verbinding maken met de back-end.

![Voorbeeld van de test Application gateway][1]

Naast het gebruik van standaard health test controle, kunt u ook de health test aanpassen aan de vereisten van uw toepassing aanpassen. In dit artikel, zowel standaard en aangepaste statuscontroles vallen.

> [!NOTE]
> Als er een NSG op Application Gateway-subnet, moeten poortbereiken 65503 65534 geopend zijn op het subnet voor Application Gateway voor binnenkomend verkeer. Deze poorten zijn vereist voor de back-end-status API om te werken.

## <a name="default-health-probe"></a>Standaard health test

Een application gateway wordt een standaard health test automatisch geconfigureerd wanneer u een aangepaste test-configuratie niet instellen. Het gedrag van de controle werkt door het maken van een HTTP-aanvraag naar de IP-adressen geconfigureerd voor de groep back-end. Voor standaard tests als de back-end-HTTP-instellingen zijn geconfigureerd voor HTTPS, de test maakt gebruik van HTTPS en status van de back-ends testen.

Bijvoorbeeld: U configureert uw toepassingsgateway voor het gebruik van back-endservers A, B en C voor het ontvangen van HTTP-verkeer op poort 80. Statuscontrole standaard test de drie servers elke 30 seconden voor een goede HTTP-antwoord. Een gezonde HTTP-antwoord heeft een [statuscode](https://msdn.microsoft.com/library/aa287675.aspx) tussen 200 en 399.

Als de standaard WebTest controle voor server A mislukt, de toepassingsgateway wordt deze verwijderd uit de back-end-pool en netwerkverkeer wordt gestopt tot deze server. De standaard-test blijft nog steeds controleren voor de server een elke 30 seconden. Als server A is op een aanvraag van een standaard health test reageert, deze weer als goed is toegevoegd aan de groep back-end en weer verkeer naar de server opnieuw.

### <a name="probe-matching"></a>Test die overeenkomt met

Standaard een HTTP (S) antwoord met statuscode 200 wordt beschouwd als in orde. Aangepaste statuscontroles ondersteunen bijkomend twee overeenkomende criteria. Die overeenkomen met criteria kan worden gebruikt om de standaardinterpretatie van welke consititutes eventueel een gezonde antwoord worden gewijzigd.

De volgende zijn die overeenkomen met criteria: 

- **HTTP-antwoord status code overeen** - test die overeenkomt met het criterium voor het accepteren van gebruiker opgegeven HTTP-antwoord code of antwoord code bereiken. Afzonderlijke door komma's gescheiden statuscodes antwoord of een bereik van statuscode wordt ondersteund.
- **HTTP-antwoord instantie overeen** - test op HTTP-antwoordtekst en overeenkomsten aan een gebruiker ziet eruit tekenreeks opgegeven vergelijkingscriterium. Merk op dat overeenkomst met de zoekt alleen voor de aanwezigheid van de gebruiker opgegeven tekenreeks in de hoofdtekst van het antwoord en is niet de overeen met een volledige reguliere expressie.

Vergelijkingscriteria kunnen worden opgegeven met de `New-AzureRmApplicationGatewayProbeHealthResponseMatch` cmdlet.

Bijvoorbeeld:

```
$match = New-AzureRmApplicationGatewayProbeHealthResponseMatch -StatusCode 200-399
$match = New-AzureRmApplicationGatewayProbeHealthResponseMatch -Body "Healthy"
```
Zodra de vergelijkingscriteria is opgegeven, deze kan worden gekoppeld als u wilt met behulp van configuratie-test een `-Match` parameter in PowerShell.

### <a name="default-health-probe-settings"></a>Standaardinstellingen health test

| Test-eigenschap | Waarde | Beschrijving |
| --- | --- | --- |
| Test-URL |http://127.0.0.1:\<Poort\>/ |URL-pad |
| Interval |30 |Testinterval in seconden |
| Time-out |30 |Test time-out in seconden |
| Drempelwaarde voor onjuiste status |3 |Aantal nieuwe pogingen-test. De back-endserver is gemarkeerd als niet actief nadat de foutentelling opeenvolgende test heeft de drempelwaarde voor onjuiste status bereikt. |

> [!NOTE]
> De poort is dezelfde poort als de back-end-HTTP-instellingen.

De standaard-test wordt bekeken alleen http://127.0.0.1: \<poort\> health-status vast te stellen. Als u nodig hebt voor het configureren van de health test om te gaan naar een aangepaste URL of andere instellingen te wijzigen, moet u aangepaste tests gebruiken, zoals beschreven in de volgende stappen uit:

## <a name="custom-health-probe"></a>Aangepaste health test

Aangepaste tests kunnen u een gedetailleerdere controle over de statuscontrole. Wanneer u aangepaste tests, kunt u de testinterval, de URL en pad voor het testen en hoeveel mislukte reacties op accepteren voordat de back-endpool-instantie als beschadigd gemarkeerd.

### <a name="custom-health-probe-settings"></a>Instellingen voor de test aangepaste status

De volgende tabel bevat de definities voor de eigenschappen van een aangepaste health test.

| Test-eigenschap | Beschrijving |
| --- | --- |
| Naam |Naam van de test. Deze naam wordt gebruikt om te verwijzen naar de instellingen voor back-end-HTTP-test. |
| Protocol |Het protocol dat wordt gebruikt voor het verzenden van de test. De test wordt gebruikt voor het protocol dat is gedefinieerd in de back-end-HTTP-instellingen |
| Host |De hostnaam voor het verzenden van de test. Van toepassing alleen als er meerdere locaties is geconfigureerd op Application Gateway, of gebruik anders '127.0.0.1'. Deze waarde verschilt van de VM-hostnaam. |
| Pad |Relatieve pad van de test. Het pad geldig wordt gestart vanuit '/'. |
| Interval |WebTest interval in seconden. Deze waarde is het interval tussen twee opeenvolgende tests. |
| Time-out |WebTest time-outwaarde in seconden. Als een geldig antwoord niet binnen deze time-outperiode ontvangen is, wordt de test als mislukt gemarkeerd.  |
| Drempelwaarde voor onjuiste status |Aantal nieuwe pogingen-test. De back-endserver is gemarkeerd als niet actief nadat de foutentelling opeenvolgende test heeft de drempelwaarde voor onjuiste status bereikt. |

> [!IMPORTANT]
> Als Application Gateway is geconfigureerd voor een enkele site, standaard ingesteld op de Host moet naam worden opgegeven als '127.0.0.1', tenzij anders is geconfigureerd in aangepaste test.
> Ter referentie een aangepaste test wordt verzonden naar \<protocol\>://\<host\>:\<poort\>\<pad\>. De poort die wordt gebruikt is dezelfde poort zoals gedefinieerd in de back-end-HTTP-instellingen.

## <a name="next-steps"></a>Volgende stappen
Nadat de informatie over Application Gateway statuscontrole, kunt u configureren een [aangepaste health test](application-gateway-create-probe-portal.md) in de Azure portal of een [aangepaste health test](application-gateway-create-probe-ps.md) met PowerShell en de Azure Resource Manager-implementatiemodel.

[1]: ./media/application-gateway-probe-overview/appgatewayprobe.png
