---
title: Taken verdelen over virtuele Windows-machines in Azure | Microsoft Docs
description: "Leer hoe u de load balancer van Azure kunt gebruiken om een maximaal beschikbare en veilige toepassing te creëren op drie Windows VM’s"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: iainfoulds
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: f0e154d0ac917d2ef2799431a72969a96415e0c0
ms.sourcegitcommit: 95500c068100d9c9415e8368bdffb1f1fd53714e
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 02/14/2018
---
# <a name="how-to-load-balance-windows-virtual-machines-in-azure-to-create-a-highly-available-application"></a>Taken verdelen tussen virtuele Windows-machines in Azure om een maximaal beschikbare toepassing te maken
Taakverdeling zorgt voor een hogere beschikbaarheid door binnenkomende aanvragen te spreiden over meerdere virtuele machines. In deze zelfstudie leert u meer over de verschillende onderdelen van Azure Load Balancer die het verkeer verdelen en zorgen voor hoge beschikbaarheid. Procedures voor:

> [!div class="checklist"]
> * Een Azure-load balancer maken
> * Een load balancer-statustest maken
> * Load balancer-verkeersregels maken
> * De Custom Script Extension gebruiken om een eenvoudige IIS-site te maken
> * Virtuele machines maken en koppelen aan een load balancer
> * Een load balancer in actie zien
> * VM's toevoegen aan en verwijderen uit een load balancer

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

Als u PowerShell lokaal wilt installeren en gebruiken, is voor deze zelfstudie moduleversie 5.3 of later van Azure PowerShell vereist. Voer `Get-Module -ListAvailable AzureRM` uit om de versie te bekijken. Als u PowerShell wilt upgraden, raadpleegt u [De Azure PowerShell-module installeren](/powershell/azure/install-azurerm-ps). Als u PowerShell lokaal uitvoert, moet u ook `Login-AzureRmAccount` uitvoeren om verbinding te kunnen maken met Azure. 


## <a name="azure-load-balancer-overview"></a>Overzicht van Azure Load Balancer
Een Azure-load balancer is een Layer-4-load balancer (TCP, UDP) die zorgt voor hoge beschikbaarheid door inkomend verkeer te distribueren tussen correct werkende virtuele machines. Tijdens een statustest van een load balancer wordt een bepaalde poort op elke VM gecontroleerd en wordt verkeer alleen naar een werkende VM gedistribueerd.

U definieert een front-end-IP-configuratie met een of meer openbare IP-adressen. Dankzij deze front-end-IP-configuratie zijn uw load balancer en toepassingen toegankelijk via internet. 

Virtuele machines maken verbinding met een load balancer via hun virtuele netwerkinterfacekaart (NIC). Om verkeer te distribueren naar de VM's bevat een back-end-adresgroep de IP-adressen van de virtuele netwerkinterfacekaarten (NIC's) die zijn verbonden met de load balancer.

Om de verkeersstroom te regelen, definieert u load balancer-regels voor specifieke poorten en protocollen die aan uw VM's worden toegewezen.


## <a name="create-azure-load-balancer"></a>Azure-load balancer maken
In dit gedeelte wordt beschreven hoe u elk onderdeel van de load balancer kunt maken en configureren. Voordat u een load balancer kunt maken, moet u eerst een resourcegroep maken met [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup). In het volgende voorbeeld wordt een resourcegroep met de naam *myResourceGroupLoadBalancer* gemaakt op de locatie *VS Oost*:

```azurepowershell-interactive
New-AzureRmResourceGroup `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS"
```

### <a name="create-a-public-ip-address"></a>Een openbaar IP-adres maken
Om toegang te krijgen tot uw app op internet, hebt u een openbaar IP-adres nodig voor de load balancer. Maak een openbaar IP-adres met [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress). In het volgende voorbeeld wordt een openbaar IP-adres met de naam *myPublicIP* gemaakt in de resourcegroep *myResourceGroupLoadBalancer*:

```azurepowershell-interactive
$publicIP = New-AzureRmPublicIpAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -AllocationMethod "Static" `
  -Name "myPublicIP"
```

### <a name="create-a-load-balancer"></a>Een load balancer maken
Maak een front-end-IP-adresgroep met [New-AzureRmLoadBalancerFrontendIpConfig](/powershell/module/azurerm.network/new-azurermloadbalancerfrontendipconfig). In het volgende voorbeeld wordt een front-end-IP-groep met de naam *myFrontEndPool* gemaakt en wordt het adres *myPublicIP* eraan gekoppeld: 

```azurepowershell-interactive
$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
```

Maak een back-end-adresgroep met [New-AzureRmLoadBalancerBackendAddressPoolConfig](/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig). In de resterende stappen worden de VM’s aan deze back-end-groep gekoppeld. In het volgende voorbeeld wordt een back-end-adresgroep met de naam *myBackEndPool* gemaakt:

```azurepowershell-interactive
$backendPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
```

Maak nu de load balancer met [New-AzureRmLoadBalancer](/powershell/module/azurerm.network/new-azurermloadbalancer). In het volgende voorbeeld wordt een load balancer met de naam *myLoadBalancer* gemaakt met behulp van de front-end- en back-end-IP-adresgroepen die in de voorgaande stappen zijn gemaakt:

```azurepowershell-interactive
$lb = New-AzureRmLoadBalancer `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myLoadBalancer" `
  -Location "EastUS" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool
```

### <a name="create-a-health-probe"></a>Een statustest maken
U gebruikt een statustest om de load balancer de status van uw app te laten bewaken. De statustest voegt dynamisch VM's toe aan de load balancer-rotatie of verwijdert ze, op basis van hun reactie op statuscontroles. Standaard wordt een VM uit de load balancer-distributie verwijderd na twee opeenvolgende fouten met intervallen van 15 seconden. U maakt een statustest op basis van een protocol of een specifieke statuscontrolepagina voor uw app. 

In het volgende voorbeeld wordt een TCP-test gemaakt. U kunt ook aangepaste HTTP-tests maken voor meer fijnmazige statuscontroles. Wanneer u een aangepaste HTTP-test maakt, moet u de statustestpagina maken, zoals *healthcheck.aspx*. De test moet het antwoord **HTTP 200 OK** voor de load balancer retourneren om de host in rotatie te houden.

U maakt een TCP-statustest met behulp van [Add-AzureRmLoadLoadBalancerProbeConfig](/powershell/module/azurerm.network/add-azurermloadbalancerprobeconfig). In het volgende voorbeeld wordt een statustest gemaakt met de naam *myHealthProbe*, die elke VM bewaakt op *TCP*-poort *80*:

```azurepowershell-interactive
Add-AzureRmLoadBalancerProbeConfig `
  -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
```

Werk de load balancer bij met [Set-AzureRmLoadBalancer](/powershell/module/azurerm.network/set-azurermloadbalancer) om de statustest toe te passen:

```azurepowershell-interactive
Set-AzureRmLoadBalancer -LoadBalancer $lb
```

### <a name="create-a-load-balancer-rule"></a>Een load balancer-regel maken
Een load balancer-regel wordt gebruikt om de verdeling van het verkeer over de VM's te definiëren. U definieert de front-end-IP-configuratie voor het inkomende verkeer en de back-end-IP-groep om het verkeer te ontvangen, samen met de gewenste bron- en doelpoort. Om ervoor te zorgen dat alleen VM's met een goede status verkeer ontvangen, moet u ook de te gebruiken statustest definiëren.

Maak een load balancer-regel met [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/add-azurermloadbalancerruleconfig). In het volgende voorbeeld wordt een load balancer-regel met de naam *myLoadBalancerRule* gemaakt waarmee het verkeer op *TCP*-poort *80* wordt geregeld:

```azurepowershell-interactive
$probe = Get-AzureRmLoadBalancerProbeConfig -LoadBalancer $lb -Name "myHealthProbe"

Add-AzureRmLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe $probe
```

Werk de load balancer bij met [Set-AzureRmLoadBalancer](/powershell/module/azurerm.network/set-azurermloadbalancer):

```azurepowershell-interactive
Set-AzureRmLoadBalancer -LoadBalancer $lb
```

## <a name="configure-virtual-network"></a>Virtueel netwerk configureren
Voordat u enkele VM's implementeert en uw balancer test, maakt u de ondersteunende virtuele-netwerkbronnen. Zie de zelfstudie [Manage Azure Virtual Networks](tutorial-virtual-network.md) (Virtuele Azure-netwerken beheren) voor meer informatie over virtuele netwerken.

### <a name="create-network-resources"></a>Netwerkbronnen maken
Maak een virtueel netwerk met [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork). In het volgende voorbeeld wordt een virtueel netwerk gemaakt met de naam *myVnet* met *mySubnet*:

```azurepowershell-interactive
# Create subnet config
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 192.168.1.0/24

# Create the virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Location "EastUS" `
  -Name "myVnet" `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig
```

Er worden virtuele NIC’s gemaakt met [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface). In het volgende voorbeeld worden drie virtuele NIC's gemaakt. (Eén virtuele NIC voor elke VM die u in de volgende stappen voor uw app maakt). U kunt op elk gewenst moment extra virtuele NIC's en VM's maken en toevoegen aan de load balancer:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   New-AzureRmNetworkInterface `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -Name myVM$i `
     -Location "EastUS" `
     -Subnet $vnet.Subnets[0] `
     -LoadBalancerBackendAddressPool $lb.BackendAddressPools[0]
}
```


## <a name="create-virtual-machines"></a>Virtuele machines maken
Verbeter de hoge beschikbaarheid van uw app door uw VM's in een beschikbaarheidsset te plaatsen.

Maak een beschikbaarheidsset met [New-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/new-azurermavailabilityset). In het volgende voorbeeld wordt een beschikbaarheidsset met de naam *myAvailabilitySet* gemaakt:

```azurepowershell-interactive
$availabilitySet = New-AzureRmAvailabilitySet `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myAvailabilitySet" `
  -Location "EastUS" `
  -Sku aligned `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2
```

Stel een beheerdersnaam en -wachtwoord voor de VM’s in met [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Nu kunt u de VM’s maken met [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm). In het volgende voorbeeld worden drie VM’s en de vereiste virtuele netwerkonderdelen gemaakt als deze nog niet bestaan:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
    New-AzureRmVm `
        -ResourceGroupName "myResourceGroupLoadBalancer" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -OpenPorts 80 `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred `
        -AsJob
}
```

Door de parameter `-AsJob` wordt de VM gemaakt als achtergrondtaak, zodat u weer terugkeert naar de PowerShell-prompts. U kunt details van achtergrondtaken bekijken met de cmdlet `Job`. Het duurt enkele minuten om alle VM’s te maken en te configureren.


### <a name="install-iis-with-custom-script-extension"></a>IIS installeren met Custom Script Extension
In een eerdere zelfstudie over [Het aanpassen van een virtuele Windows-machine](tutorial-automate-vm-deployment.md) hebt u geleerd hoe u het aanpassen van VM’s kunt automatiseren met de Custom Script Extension voor Windows. U kunt dezelfde benadering gebruiken om IIS op uw VM’s te installeren en te configureren.

Gebruik [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) om de aangepaste scriptextensie te installeren. De extensie voert `powershell Add-WindowsFeature Web-Server` uit om de IIS-webserver te installeren en werkt vervolgens de pagina *Default.htm* bij om de hostnaam van de VM weer te geven:

```azurepowershell-interactive
for ($i=1; $i -le 3; $i++)
{
   Set-AzureRmVMExtension `
     -ResourceGroupName "myResourceGroupLoadBalancer" `
     -ExtensionName "IIS" `
     -VMName myVM$i `
     -Publisher Microsoft.Compute `
     -ExtensionType CustomScriptExtension `
     -TypeHandlerVersion 1.8 `
     -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
     -Location EastUS
}
```

## <a name="test-load-balancer"></a>Load balancer testen
Haal het openbare IP-adres van uw load balancer op met [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress). In het volgende voorbeeld wordt het IP-adres opgehaald voor het eerder gemaakte *myPublicIP*:

```azurepowershell-interactive
Get-AzureRmPublicIPAddress `
  -ResourceGroupName "myResourceGroupLoadBalancer" `
  -Name "myPublicIP" | select IpAddress
```

Vervolgens kunt u het openbare IP-adres invoeren in een webbrowser. De website wordt weergegeven met de hostnaam van de VM waarnaar de load balancer verkeer heeft gedistribueerd, zoals in het volgende voorbeeld:

![Actieve IIS-website](./media/tutorial-load-balancer/running-iis-website.png)

Als u wilt zien hoe de load balancer verkeer distribueert naar alle drie de VM's waarop uw app wordt uitgevoerd, kunt u vernieuwing van uw webbrowser afdwingen.


## <a name="add-and-remove-vms"></a>VM's toevoegen en verwijderen
Het is mogelijk dat u onderhoud moet uitvoeren op de VM's waarop uw app wordt uitgevoerd, zoals het installeren van besturingssysteemupdates. U moet mogelijk extra VM's toevoegen vanwege toegenomen verkeer naar uw app. In dit gedeelte wordt beschreven hoe u een VM aan de load balancer toevoegt of ervan verwijdert.

### <a name="remove-a-vm-from-the-load-balancer"></a>Een VM van de load balancer verwijderen
Haal de netwerkinterfacekaart op met [Get-AzureRmNetworkInterface](/powershell/module/azurerm.network/get-azurermnetworkinterface) en stel de eigenschap *LoadBalancerBalancerBackendAddressPools* van de virtuele NIC in op *$null*. Werk tot slot de virtuele NIC bij:

```azurepowershell-interactive
$nic = Get-AzureRmNetworkInterface `
    -ResourceGroupName "myResourceGroupLoadBalancer" `
    -Name "myVM2"
$nic.Ipconfigurations[0].LoadBalancerBackendAddressPools=$null
Set-AzureRmNetworkInterface -NetworkInterface $nic
```

Als u wilt zien hoe de load balancer verkeer distribueert naar de resterende twee VM's waarop uw app wordt uitgevoerd, kunt u vernieuwing van uw webbrowser afdwingen. U kunt nu onderhoud uitvoeren op de VM, zoals het installeren van updates voor het besturingssysteem of het opnieuw opstarten van de VM.

### <a name="add-a-vm-to-the-load-balancer"></a>Een VM toevoegen aan de load balancer
Stel na het uitvoeren van VM-onderhoud, of als u capaciteit moet uitbreiden, de eigenschap *LoadBalancerBackendAddressPools* van het virtuele NIC in op de *BackendAddressPool* van [Get-AzureRMLoadBalancer](/powershell/module/azurerm.network/get-azurermloadbalancer):

Haal de load balancer op:

```azurepowershell-interactive
$lb = Get-AzureRMLoadBalancer `
    -ResourceGroupName myResourceGroupLoadBalancer `
    -Name myLoadBalancer 
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$lb.BackendAddressPools[0]
Set-AzureRmNetworkInterface -NetworkInterface $nic
```

## <a name="next-steps"></a>Volgende stappen

In deze zelfstudie hebt u een load balancer gemaakt en er VM's aan gekoppeld. U hebt geleerd hoe u:

> [!div class="checklist"]
> * Een Azure-load balancer maken
> * Een load balancer-statustest maken
> * Load balancer-verkeersregels maken
> * De Custom Script Extension gebruiken om een eenvoudige IIS-site te maken
> * Virtuele machines maken en koppelen aan een load balancer
> * Een load balancer in actie zien
> * VM's toevoegen aan en verwijderen uit een load balancer

Ga door naar de volgende zelfstudie voor informatie over het beheren van VM-netwerken.

> [!div class="nextstepaction"]
> [Virtuele machines en virtuele netwerken beheren](./tutorial-virtual-network.md)
