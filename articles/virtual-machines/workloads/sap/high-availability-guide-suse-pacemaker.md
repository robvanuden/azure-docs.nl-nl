---
title: Instellen van pacemaker heeft op SUSE Linux Enterprise Server in Azure | Microsoft Docs
description: Instellen van pacemaker heeft op SUSE Linux Enterprise Server in Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: mssedusch
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/20/2018
ms.author: sedusch
ms.openlocfilehash: 8db6857ba40f128c78e42eedba80e0d0062bbb27
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="setting-up-pacemaker-on-suse-linux-enterprise-server-in-azure"></a>Instellen van pacemaker heeft op SUSE Linux Enterprise Server in Azure

[planning-guide]:planning-guide.md
[deployment-guide]:deployment-guide.md
[dbms-guide]:dbms-guide.md
[sap-hana-ha]:sap-hana-high-availability.md

Er zijn twee opties voor het instellen van een cluster pacemaker heeft in Azure. U kunt een agent hekwerk, die zorgt voor opnieuw starten van een knooppunt via de Azure-API's gebruiken of kunt u een apparaat SBD.

Het apparaat SBD moet één extra virtuele machine die fungeert als een iSCSI-doelserver en biedt een SBD-apparaat. Deze iSCSI-doelserver kan echter worden gedeeld met andere clusters pacemaker heeft. Het voordeel van een apparaat SBD is een snellere failover en als u van SBD apparaten on-premises gebruikmaakt vereist geen wijzigingen op de werking van het cluster pacemaker heeft. De hekwerk SBD kunt blijven gebruiken voor de agent Azure fence als een back-up mechanisme afrasteringen als de iSCSI-doelserver niet beschikbaar is.

Als u niet investeren in één extra virtuele machine wilt, kunt u ook de Fence voor Azure-agent. Het nadeel is dat een failover tussen 10 tot 15 minuten uitvoeren kunt als een resource stoppen is mislukt of de clusterknooppunten kunnen niet die elkaar meer communiceren.

![Pacemaker heeft op SLES-overzicht](./media/high-availability-guide-suse-pacemaker/pacemaker.png)

## <a name="sbd-fencing"></a>SBD hekwerk

Volg deze stappen als u wilt een apparaat SBD voor hekwerk gebruiken.

### <a name="set-up-an-iscsi-target-server"></a>Instellen van een iSCSI-doelserver

U moet eerst een iSCSI-doel-virtuele machine maken als u geen abonnement al hebt nog. iSCSI-doelservers kunnen worden gedeeld met meerdere pacemaker heeft clusters.

1. Implementeer een nieuwe SLES 12 SP1 of hoger virtuele machine en verbinding maken met de machine via ssh. De computer hoeft niet te groot. De grootte van een virtuele machine zoals Standard_E2s_v3 of Standard_D2s_v3 is voldoende.

1. SLES bijwerken

   <pre><code>
   sudo zypper update
   </code></pre>

1. ISCSI-doel pakketten installeren

   <pre><code>
   sudo zypper install targetcli-fb dbus-1-python
   </code></pre>

1. De iSCSI-doelservice inschakelen

   <pre><code>   
   sudo systemctl enable target
   sudo systemctl enable targetcli
   sudo systemctl start target
   sudo systemctl start targetcli
   </code></pre>

### <a name="create-iscsi-device-on-iscsi-target-server"></a>ISCSI-apparaat op de iSCSI-doelserver maken

Een nieuwe gegevensschijf koppelen aan de iSCSI-doel-virtuele machine die kunnen worden gebruikt voor dit cluster. De gegevensschijf kan zo klein 1 GB en moet worden geplaatst op een Premium Storage-Account of een Premium beheerd schijf profiteren van de [enkele VM SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines).

Voer de volgende opdracht op de **iSCSI-doel VM** voor het maken van een iSCSI-schijf voor het nieuwe cluster. In het volgende voorbeeld **cl1** wordt gebruikt om het nieuwe cluster te identificeren en **prod-cl1-0** en **prod-cl1-1** de hostnamen van de clusterknooppunten zijn. Vervang ze door de hostnamen van de clusterknooppunten.

<pre><code>
# List all data disks with the following command
sudo ls -al /dev/disk/azure/scsi1/

# total 0
# drwxr-xr-x 2 root root  80 Mar 26 14:42 .
# drwxr-xr-x 3 root root 160 Mar 26 14:42 ..
# lrwxrwxrwx 1 root root  12 Mar 26 14:42 lun0 -> ../../../<b>sdc</b>
# lrwxrwxrwx 1 root root  12 Mar 26 14:42 lun1 -> ../../../sdd

# Then use the disk name to list the disk id
sudo ls -l /dev/disk/by-id/scsi-* | grep sdc

# lrwxrwxrwx 1 root root  9 Mar 26 14:42 /dev/disk/by-id/scsi-14d53465420202020a50923c92babda40974bef49ae8828f0 -> ../../sdc
# lrwxrwxrwx 1 root root  9 Mar 26 14:42 <b>/dev/disk/by-id/scsi-360022480a50923c92babef49ae8828f0 -> ../../sdc</b>

# Use the data disk that you attached for this cluster to create a new backstore
sudo targetcli backstores/block create <b>cl1</b> <b>/dev/disk/by-id/scsi-360022480a50923c92babef49ae8828f0</b>

sudo targetcli iscsi/ create iqn.2006-04.<b>cl1</b>.local:<b>cl1</b>
sudo targetcli iscsi/iqn.2006-04.<b>cl1</b>.local:<b>cl1</b>/tpg1/luns/ create /backstores/block/<b>cl1</b>
sudo targetcli iscsi/iqn.2006-04.<b>cl1</b>.local:<b>cl1</b>/tpg1/acls/ create iqn.2006-04.<b>prod-cl1-0.local:prod-cl1-0</b>
sudo targetcli iscsi/iqn.2006-04.<b>cl1</b>.local:<b>cl1</b>/tpg1/acls/ create iqn.2006-04.<b>prod-cl1-1.local:prod-cl1-1</b>

# save the targetcli changes
sudo targetcli saveconfig
sudo systemctl restart target
</code></pre>

### <a name="set-up-sbd-device"></a>SBD apparaat instellen

Verbinding maken met het iSCSI-apparaat dat is gemaakt in de laatste stap van het cluster.
Voer de volgende opdrachten op de knooppunten van het nieuwe cluster dat u wilt maken.
De volgende items worden voorafgegaan door een **[A]** - van toepassing op alle knooppunten **[1]** - alleen van toepassing op knooppunt 1 of **[2]** - alleen van toepassing op knooppunt 2.

1. **[A]**  Verbinding maken met iSCSI-apparaten

   Schakel eerst de iSCSI- en SBD services.

   <pre><code>
   sudo systemctl enable iscsid
   sudo systemctl enable iscsi
   sudo systemctl enable sbd
   </code></pre>

1. **[1]**  Wijzig de initiatornaam van de op het eerste knooppunt

   <pre><code>
   sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   De inhoud van het bestand dat moet overeenkomen met de ACL's die u hebt gebruikt bij het maken van het iSCSI-apparaat op de iSCSI-doelserver wijzigen

   <pre><code>   
   InitiatorName=<b>iqn.2006-04.prod-cl1-0.local:prod-cl1-0</b>
   </code></pre>

1. **[2]**  Wijzig de initiatornaam van de in het tweede knooppunt

   <pre><code>
   sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   De inhoud van het bestand dat moet overeenkomen met de ACL's die u hebt gebruikt bij het maken van het iSCSI-apparaat op de iSCSI-doelserver wijzigen

   <pre><code>
   InitiatorName=<b>iqn.2006-04.prod-cl1-1.local:prod-cl1-1</b>
   </code></pre>

1. **[A]**  De iSCSI-service opnieuw starten

   Nu opnieuw opstarten van de iSCSI-service als de wijziging wilt toepassen
   
   <pre><code>
   sudo systemctl restart iscsid
   sudo systemctl restart iscsi
   </code></pre>

   Verbinding maken met iSCSI-apparaten. In het onderstaande voorbeeld 10.0.0.17 is het IP-adres van de iSCSI-doelserver en 3260 is de standaardpoort. <b>IQN.2006 04.cl1.local:cl1</b> is de naam van het doel dat wordt weergegeven als u de eerste opdracht uitvoert.

   <pre><code>
   sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.17:3260</b>
   
   sudo iscsiadm -m node -T <b>iqn.2006-04.cl1.local:cl1</b> --login --portal=<b>10.0.0.17:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.17:3260</b> --op=update --name=node.startup --value=automatic
   </code></pre>

   Zorg ervoor dat de iSCSI-apparaat beschikbaar is en noteer de naam van het apparaat (in het volgende voorbeeld/dev/sde) uitgevoerd

   <pre><code>
   lsscsi
   
   # [2:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sda
   # [3:0:1:0]    disk    Msft     Virtual Disk     1.0   /dev/sdb
   # [5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
   # [5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd
   # <b>[6:0:0:0]    disk    LIO-ORG  cl1              4.0   /dev/sde</b>
   </code></pre>

   De ID van het iSCSI-apparaat nu ophalen.

   <pre><code>
   ls -l /dev/disk/by-id/scsi-* | grep <b>sde</b>
   
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-1LIO-ORG_cl1:3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   # <b>lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df -> ../../sde</b>
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-SLIO-ORG_cl1_3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   </code></pre>

   De opdracht drie apparaat-id's weergeven. Het is raadzaam het gebruik van de ID die met de scsi-3, in het bovenstaande dit voorbeeld begint is
   
   **/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df**

1. **[1]**  De SBD apparaat maken

   Gebruik de apparaat-ID van het iSCSI-apparaat te maken van een nieuw SBD apparaat op het eerste clusterknooppunt.

   <pre><code>
   sudo sbd -d <b>/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df</b> -1 10 -4 20 create
   </code></pre>

1. **[A]**  SBD config aanpassen

   Het configuratiebestand SBD openen

   <pre><code>
   sudo vi /etc/sysconfig/sbd
   </code></pre>

   Wijzig de eigenschap van het apparaat SBD, de integratie pacemaker heeft en de startmodus van SBD wijzigen.

   <pre><code>
   [...]
   <b>SBD_DEVICE="/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df"</b>
   [...]
   <b>SBD_PACEMAKER="yes"</b>
   [...]
   <b>SBD_STARTMODE="always"</b>
   </code></pre>

   Het configuratiebestand softdog maken

   <pre><code>
   echo softdog | sudo tee /etc/modules-load.d/softdog.conf
   </code></pre>

   Nu de module niet laden

   <pre><code>
   sudo modprobe -v softdog
   </code></pre>

## <a name="cluster-installation"></a>Clusterinstallatie

De volgende items worden voorafgegaan door een **[A]** - van toepassing op alle knooppunten **[1]** - alleen van toepassing op knooppunt 1 of **[2]** - alleen van toepassing op knooppunt 2.

1. **[A]**  SLES bijwerken

   <pre><code>
   sudo zypper update
   </code></pre>

1. **[1]**  Ssh toegang inschakelen

   <pre><code>
   sudo ssh-keygen
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # copy the public key
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

2. **[2]**  Ssh toegang inschakelen

   <pre><code>
   sudo ssh-keygen
   
   # insert the public key you copied in the last step into the authorized keys file on the second server
   sudo vi /root/.ssh/authorized_keys
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # copy the public key   
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

1. **[1]**  Ssh toegang inschakelen

   <pre><code>
   # insert the public key you copied in the last step into the authorized keys file on the first server
   sudo vi /root/.ssh/authorized_keys
   </code></pre>

1. **[A]**  Installeren HA-uitbreiding
   
   <pre><code>
   sudo zypper install sle-ha-release fence-agents
   </code></pre>

1. **[A]**  Hostnamen instellen   

   U kunt gebruik van een DNS-server of wijzig de/etc/hosts op alle knooppunten. In dit voorbeeld laat zien hoe het bestand/etc/hosts te gebruiken.
   Vervang het IP-adres en de hostnaam in de volgende opdrachten. Het voordeel van/etc/hosts is dat uw cluster onafhankelijk van DNS die één toegangspunt fouten te worden.

   <pre><code>
   sudo vi /etc/hosts
   </code></pre>
   
   Voeg de volgende regels/etc/hosts. Wijzig de IP-adres en hostnaam overeenkomen met uw omgeving   
   
   <pre><code>
   # IP address of the first cluster node
   <b>10.0.0.6 prod-cl1-0</b>
   # IP address of the second cluster node
   <b>10.0.0.7 prod-cl1-1</b>
   </code></pre>

1. **[1]**  Cluster installeren
   
   <pre><code>
   sudo ha-cluster-init
   
   # Do you want to continue anyway? [y/N] -> y
   # Network address to bind to (for example: 192.168.1.0) [10.79.227.0] -> Press ENTER
   # Multicast address (for example: 239.x.x.x) [239.174.218.125] -> Press ENTER
   # Multicast port [5405] -> Press ENTER
   # Do you wish to configure an administration IP? [y/N] -> N
   </code></pre>

1. **[2]**  Knooppunt aan cluster toevoegen
   
   <pre><code> 
   sudo ha-cluster-join
   
   # Do you want to continue anyway? [y/N] -> y
   # IP address or hostname of existing node (for example: 192.168.1.1) [] -> IP address of node 1 for example 10.0.0.14
   # /root/.ssh/id_rsa already exists - overwrite? [y/N] N
   </code></pre>

1. **[A]**  Hacluster wachtwoord wijzigen naar hetzelfde wachtwoord

   <pre><code> 
   sudo passwd hacluster
   </code></pre>

1. **[A]**  Corosync voor het gebruik van andere transport en voeg nodelist configureren. Cluster werkt niet anders.
   
   <pre><code> 
   sudo vi /etc/corosync/corosync.conf   
   </code></pre>

   De volgende vet inhoud toevoegen aan het bestand als de waarden niet er of andere.
   
   <pre><code> 
   [...]
     <b>token:          5000
     token_retransmits_before_loss_const: 10
     join:           60
     consensus:      6000
     max_messages:   20</b>
     
     interface { 
        [...] 
     }
     <b>transport:      udpu</b>
   } 
   <b>nodelist {
     node {
      # IP address of <b>prod-cl1-0</b>
      ring0_addr:10.0.0.6
     }
     node {
      # IP address of <b>prod-cl1-1</b>
      ring0_addr:10.0.0.7
     } 
   }</b>
   logging {
     [...]
   }
   quorum {
        # Enable and configure quorum subsystem (default: off)
        # see also corosync.conf.5 and votequorum.5
        provider: corosync_votequorum
        <b>expected_votes: 2</b>
        <b>two_node: 1</b>
   }
   </code></pre>

   Start de service corosync opnieuw

   <pre><code>
   sudo service corosync restart
   </code></pre>

1. **[1]**  De pacemaker heeft standaardinstellingen wijzigen

   <pre><code>
   sudo crm configure rsc_defaults resource-stickiness="1"   
   </code></pre>

## <a name="create-stonith-device"></a>STONITH apparaat maken

Het apparaat STONITH maakt gebruik van een Service-Principal worden geautoriseerd op basis van Microsoft Azure. Volg deze stappen voor het maken van een Service-Principal.

1. Ga naar <https://portal.azure.com>
1. Open de blade van Azure Active Directory  
   Ga naar eigenschappen en noteer de Directory-ID. Dit is de **tenant-ID**.
1. Klik op App-registraties
1. Klik op Add.
1. Voer een naam, selecteert u toepassingstype 'Web-app /-API, voer een URL voor eenmalige aanmelding (bijvoorbeeld http://localhost) en klik op maken
1. De aanmeldings-URL kan niet wordt gebruikt en geldige URL
1. Selecteer de nieuwe App en sleutels op in het tabblad instellingen
1. Voer een beschrijving voor een nieuwe sleutel, selecteer 'Verloopt nooit' en klik op Opslaan
1. Noteer de waarde in. Deze wordt gebruikt als de **wachtwoord** voor de Service-Principal
1. Noteer de aanvraag-ID. Deze wordt gebruikt als de gebruikersnaam (**aanmeldings-ID** in de onderstaande stappen) van de Service-Principal

### <a name="1-create-a-custom-role-for-the-fence-agent"></a>**[1]**  Maakt u een aangepaste rol voor de agent fence

De Service-Principal heeft geen machtigingen voor toegang tot uw Azure-resources standaard. U moet de Service-Principal machtigingen geven om te starten en stoppen (ongedaan gemaakt) alle virtuele machines van het cluster. Als u de aangepaste rol die niet al hebt gemaakt, kunt u met behulp van [PowerShell](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-powershell#create-a-custom-role) of [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli#create-a-custom-role)

Gebruik de volgende inhoud voor het bestand voor invoer. U moet de inhoud aan uw abonnementen, die is aangepast, c276fc76-9cd4-44c9-99a7-4fd71546436e en e91d47c4-76f3-4271-a796-21b4ecfe3624 vervangen door de id's van uw abonnement. Als u slechts één abonnement hebt, verwijdert u de tweede vermelding in AssignableScopes.

```json
{
  "Name": "Linux Fence Agent Role",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to deallocate and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
    "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"
  ]
}
```

### <a name="1-assign-the-custom-role-to-the-service-principal"></a>**[1]**  De aangepaste rol toewijzen aan de Service-Principal

De aangepaste rol 'Linux Fence Agent rol' die is gemaakt in het vorige hoofdstuk naar de Service-Principal toe te wijzen. Gebruik de rol van eigenaar niet meer!

1. Ga naar https://portal.azure.com
1. Open de blade met alle bronnen
1. Selecteer de virtuele machine van het eerste clusterknooppunt
1. Klik op toegangsbeheer (IAM)
1. Klik op Add.
1. Selecteer de rol 'Linux Fence Agent rol'
1. Voer de naam van de toepassing die u hierboven hebt gemaakt
1. Klik op OK

Herhaal de bovenstaande stappen voor het tweede clusterknooppunt.

### <a name="1-create-the-stonith-devices"></a>**[1]**  De STONITH apparaten maken

Nadat u de machtigingen voor de virtuele machines hebt bewerkt, kunt u de apparaten STONITH configureren in het cluster.

<pre><code>
# replace the bold string with your subscription ID, resource group, tenant ID, service principal ID and password
sudo crm configure property stonith-timeout=900

sudo crm configure primitive rsc_st_azure stonith:fence_azure_arm \
   params subscriptionId="<b>subscription ID</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" login="<b>login ID</b>" passwd="<b>password</b>"

</code></pre>

### <a name="1-create-fence-topology-for-sbd-fencing"></a>**[1]**  Fence topologie voor SBD hekwerk maken

Als u SBD-apparaat gebruikt wilt, nog beter een Azure fence agent als een back-up als de iSCSI-doelserver niet beschikbaar is.

<pre><code>
sudo crm configure fencing_topology \
  stonith-sbd rsc_st_azure

</code></pre>
### **[1] ** Het gebruik van een apparaat STONITH inschakelen

<pre><code>
sudo crm configure property stonith-enabled=true 
</code></pre>


## <a name="next-steps"></a>Volgende stappen
* [Azure virtuele Machines, planning en implementatie voor SAP][planning-guide]
* [Azure virtuele Machines-implementatie voor SAP][deployment-guide]
* [Azure virtuele Machines DBMS-implementatie voor SAP][dbms-guide]
* Zie voor meer informatie over hoe u hoge beschikbaarheid en herstel na noodgevallen van SAP HANA plannen op Azure Virtual machines, [hoge beschikbaarheid van SAP HANA op Azure Virtual Machines (VM's)][sap-hana-ha]
