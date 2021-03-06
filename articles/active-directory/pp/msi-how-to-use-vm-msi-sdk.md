---
title: Het gebruik van een gebruiker toegewezen beheerde Service-identiteit van de Azure SDK's op een virtuele machine
description: Codevoorbeelden voor het gebruik van Azure SDK's met een MSI gebruiker toegewezen op een virtuele machine.
services: active-directory
documentationcenter: 
author: daveba
manager: mtillman
editor: 
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/22/2017
ms.author: daveba
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 808b0357494adac8c8ad7f51e668317e378d290d
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 02/24/2018
---
# <a name="use-azure-sdks-with-a-user-assigned-managed-service-identity-msi"></a>Gebruik Azure SDK's met een gebruiker toegewezen beheerde Service identiteit (MSI)

[!INCLUDE[preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

Dit artikel bevat een lijst met de SDK-voorbeelden die gebruik van hun respectieve Azure SDK van ondersteuning voor de gebruiker toegewezen MSI aantonen.

## <a name="prerequisites"></a>Vereisten

[!INCLUDE [msi-core-prereqs](~/includes/active-directory-msi-core-prereqs-ua.md)]

> [!IMPORTANT]
> - Alle code/voorbeeldscript in dit artikel wordt ervan uitgegaan dat de client wordt uitgevoerd op een virtuele Machine van MSI-functionaliteit. Gebruik de virtuele machine 'Verbinding'-functie in de Azure portal op afstand verbinding maken met uw virtuele machine. Zie voor meer informatie over het inschakelen van MSI op een virtuele machine [configureren van een VM beheerde Service identiteit (MSI) met de Azure CLI](msi-qs-configure-cli-windows-vm.md), of een van de variant artikelen (met behulp van PowerShell, Azure-portal, een sjabloon of een Azure-SDK). 

## <a name="sdk-code-samples"></a>SDK-codevoorbeelden

| SDK             | Codevoorbeeld |
| --------------- | ----------- |
| Python          | [MSI gebruiken om te verifiëren gewoon uit in een virtuele machine](https://azure.microsoft.com/resources/samples/resource-manager-python-manage-resources-with-msi/) |
| Ruby            | [Resources beheren van een VM MSI-ingeschakeld](https://azure.microsoft.com/resources/samples/resources-ruby-manage-resources-with-msi/) |

## <a name="next-steps"></a>Volgende stappen

- Zie [Azure SDK's](https://azure.microsoft.com/downloads/) voor een volledige lijst met Azure SDK-bronnen, met inbegrip van bibliotheek downloads, documentatie en meer.

Gebruik de volgende sectie met opmerkingen uw feedback en help ons verfijnen en onze content vorm.








