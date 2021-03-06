---
title: 'Azure CLI-voorbeeldscript: een beheerde schijf maken op basis van een VHD-bestand in een opslagaccount in hetzelfde abonnement | Microsoft Docs'
description: 'Azure CLI-voorbeeldscript: een beheerde schijf maken op basis van een VHD-bestand in een opslagaccount in hetzelfde abonnement'
services: virtual-machines-linux
documentationcenter: storage
author: ramankumarlive
manager: kavithag
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: 15a5e3e566b03f29f33c3f21aa6581a03b0abe2e
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/09/2018
---
# <a name="create-a-managed-disk-from-a-vhd-file-in-a-storage-account-in-the-same-subscription-with-cli"></a>Maak een beheerde schijf op basis van een VHD-bestand in een opslagaccount in hetzelfde abonnement met CLI

Met dit script maakt u een beheerde schijf op basis van een VHD-bestand in een opslagaccount in hetzelfde abonnement. Gebruik dit script voor het importeren van een gespecialiseerde (geen gegeneraliseerde/op het systeem voorbereide) VHD naar een beheerde OS-schijf voor het maken van een virtuele machine. Of gebruik het script om een gegevens-VHD naar een beheerde gegevensschijf te importeren. 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Voorbeeldscript

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-managed-data-disks-from-vhd/create-managed-data-disks-from-vhd.sh "Create managed disk from VHD")]


## <a name="script-explanation"></a>Uitleg van het script

Dit script gebruikt de volgende opdrachten om een beheerde schijf te maken op basis van een VHD. Elke opdracht in de tabel is een koppeling naar specifieke documentatie over de opdracht.

| Opdracht | Opmerkingen |
|---|---|
| [az disk create](https://docs.microsoft.com/cli/azure/disk#az_disk_create) | Hiermee maakt u een beheerde schijf met behulp van de URI van een VHD in een opslagaccount in hetzelfde abonnement. |

## <a name="next-steps"></a>Volgende stappen

[Een virtuele machine maken door een beheerde schijf te koppelen als besturingssysteemschijf](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

Raadpleeg de [documentatie van Azure CLI](https://docs.microsoft.com/cli/azure) voor meer informatie over de Azure CLI.

Aanvullende CLI-scriptvoorbeelden voor virtuele machines en beheerde schijven vindt u in de [Azure-documentatie voor Linux-VM's](../../app-service/app-service-cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
