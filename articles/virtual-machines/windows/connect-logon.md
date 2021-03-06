---
title: Verbinding maken met een virtuele Windows Server-machine | Microsoft Docs
description: Leer hoe u verbinding maakt en u aanmeldt bij een virtuele Windows-machine via de Azure-portal en het Resource Manager-implementatiemodel.
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ef62b02e-bf35-468d-b4c3-71b63fe7f409
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/11/2018
ms.author: cynthn
ms.openlocfilehash: 0c81e70a76983885fdfb6eefe9b6cbe407e117c8
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="how-to-connect-and-log-on-to-an-azure-virtual-machine-running-windows"></a>Verbinding maken met en aanmelden bij een virtuele machine in Azure waarop Windows wordt uitgevoerd
U gebruikt de knop **Verbinden** in Azure Portal om een Extern bureaublad-sessie (RDP) te starten vanaf een Windows-computer. Eerst maakt u verbinding met de virtuele machine en vervolgens meldt u zich aan.

Als u vanaf een Mac probeert verbinding te maken met een virtuele Windows-machine, moet u een RDP-client voor Mac installeren, zoals [Microsoft Extern bureaublad](https://itunes.apple.com/app/microsoft-remote-desktop/id715768417).

## <a name="connect-to-the-virtual-machine"></a>Verbinding maken met de virtuele machine
1. Meld u aan bij de [Azure-portal](https://portal.azure.com/) als u dat nog niet hebt gedaan.
2. Klik in het menu links op **Virtuele machines**.
3. Selecteer de virtuele machine in de lijst.
4. Klik boven aan de pagina voor de virtuele machine op de ![Afbeelding van de knop verbinding maken.](./media/connect-logon/connect.png) .
   
   > [!TIP]
   > Als de knop **Verbinden** in de portal grijs is en u niet met Azure bent verbonden via een [Snelle route](../../expressroute/expressroute-introduction.md) of een [VPN-tussen-sites](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)-verbinding, moet u een openbaar IP-adres maken en toewijzen aan uw virtuele machine voordat u Extern bureaublad kunt gebruiken. Er is meer informatie over [openbare IP-adressen in Azure](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) beschikbaar.
   > 
   > 

## <a name="log-on-to-the-virtual-machine"></a>Aanmelden bij de virtuele machine
[!INCLUDE [virtual-machines-log-on-win-server](../../../includes/virtual-machines-log-on-win-server.md)]

## <a name="next-steps"></a>Volgende stappen
Zie [Problemen oplossen met Extern bureaublad-verbindingen](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) als u problemen hebt wanneer u verbinding probeert te maken. Dit artikel leidt u door het opsporen en oplossen van veelvoorkomende problemen.

