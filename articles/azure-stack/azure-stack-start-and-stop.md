---
title: Starten en stoppen van de Azure-Stack | Microsoft Docs
description: Informatie over het starten en Azure Stack afgesloten.
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 43BF9DCF-F1B7-49B5-ADC5-1DA3AF9668CA
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/09/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 53015ba5c282bbe9c7b8185b080ffb6d834b6c75
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="start-and-stop-azure-stack"></a>Starten en stoppen van de Azure-Stack
Volg de procedures in dit artikel voor het correct afsluiten en opnieuw opstarten van de Stack van het Azure-services. 

## <a name="stop-azure-stack"></a>Azure Stack stoppen 

Azure-Stack afsluiten met de volgende stappen:

1. Open een bevoorrechte eindpunt sessie (PEP) van een computer met toegang tot het netwerk naar de Azure-Stack ERCS VM's. Zie voor instructies [met behulp van de bevoegde eindpunt in Azure-Stack](azure-stack-privileged-endpoint.md).

2. Voer het volgende uit de PEP:

    ```powershell
      Stop-AzureStack
    ```

3. Wachten op alle fysieke knooppunten voor Azure-Stack tot macht uitschakelen.

> [!Note]  
> U kunt de energiestatus van een fysiek knooppunt controleren door de instructies van de Original Equipment Manufacturer (OEM) die uw Azure-Stack hardware opgegeven. 

## <a name="start-azure-stack"></a>Start Azure Stack 

Azure-Stack beginnen met de volgende stappen uit. Volg deze stappen, ongeacht hoe Azure-Stack is gestopt.

1. Schakel op alle fysieke knooppunten in uw Azure-Stack-omgeving. Controleer of u de instructies voor de fysieke knooppunten inschakelen door de instructies van de Original Equipment Manufacturer (OEM) die de hardware voor uw Azure-Stack opgegeven.

2. Wacht totdat de Stack van Azure-infrastructuurservices wordt gestart. Azure Stack-infrastructuurservices kunnen vereisen twee uur aan het beginproces is voltooid. U kunt controleren of de status van de start van Azure-Stack met de [ **Get-ActionStatus** cmdlet](#get-the-startup-status-for-azure-stack).


## <a name="get-the-startup-status-for-azure-stack"></a>Status ophalen voor het starten van de voor Azure-Stack

Haal het opstarten van de voor de Azure-Stack Opstartroutine met de volgende stappen:

1. Open een bevoorrechte Endpoint-sessie van een computer met toegang tot het netwerk naar de Azure-Stack ERCS VM's.

2. Voer het volgende uit de PEP:

    ```powershell
      Get-ActionStatus Start-AzureStack
    ```

## <a name="troubleshoot-startup-and-shutdown-of-azure-stack"></a>Problemen met opstarten en afsluiten van de Azure-Stack

De volgende stappen uitvoeren als de infrastructuur en tenant-services niet 2 uur nadat u power van uw Azure-Stack-omgeving starten. 

1. Open een bevoorrechte Endpoint-sessie van een computer met toegang tot het netwerk naar de Azure-Stack ERCS VM's.

2. Uitvoeren: 

    ```powershell
      Test-AzureStack
      ```

3. Controleer de uitvoer en los eventuele fouten health. Zie voor meer informatie [uitvoeren van een validatietest van Azure-Stack](azure-stack-diagnostic-test.md).

4. Uitvoeren:

    ```powershell
      Start-AzureStack
    ```

5. Als met **Start AzureStack** resulteert in een fout, neem contact op met de klantondersteuning van Microsoft voor Services. 

## <a name="next-steps"></a>Volgende stappen 

Meer informatie over Azure-Stack diagnostische hulpprogramma en uitgeven van logboekregistratie, Zie [diagnostische hulpprogramma's voor een Azure-Stack](azure-stack-diagnostics.md).
