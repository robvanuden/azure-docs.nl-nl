---
title: Instellen van herstel na noodgevallen in Azure voor fysieke on-premises servers met Azure Site Recovery | Microsoft Docs
description: Informatie over het instellen van herstel na noodgevallen in Azure voor lokale Windows- en Linux-servers, met de Azure Site Recovery-service.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 03/08/2018
ms.author: raynew
ms.openlocfilehash: 207ff17f7b113bf4a94bb6c157cf53e7b1c46b45
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="set-up-disaster-recovery-to-azure-for-on-premises-physical-servers"></a>Herstel na noodgevallen in Azure voor on-premises fysieke servers instellen

De [Azure Site Recovery](site-recovery-overview.md)-service draagt bij aan uw strategie voor herstel na noodgevallen door de replicatie, failover en failback van on-premises machines en virtuele Azure-machines te beheren en in te delen.

Deze zelfstudie laat zien hoe u voor het instellen van herstel na noodgevallen van on-premises fysieke Windows en Linux-servers naar Azure. In deze zelfstudie leert u het volgende:

> [!div class="checklist"]
> * Azure en on-premises vereisten instellen
> * Een Recovery Services-kluis maken voor Site Recovery 
> * Instellen van de bron en doel van de replicatie-omgevingen
> * Een replicatiebeleid maken
> * Replicatie voor een server inschakelen

## <a name="prerequisites"></a>Vereisten

Vereisten voor het voltooien van deze zelfstudie:

- Zorg ervoor dat u inzicht hebt in de [architectuur en onderdelen voor dit scenario](physical-azure-architecture.md).
- Raadpleeg de [ondersteuningsvereisten](vmware-physical-secondary-support-matrix.md) voor alle onderdelen.
- Zorg ervoor dat de servers die u wilt repliceren voldoen [vereisten van de virtuele machine van Azure](vmware-physical-secondary-support-matrix.md#replicated-vm-support).
- Voorbereiden op Azure. U moet een Azure-abonnement, een Azure-netwerk en een opslagaccount.
- Bereid een account voor automatische installatie van de Mobility-service op elke server die u wilt repliceren.

> [!NOTE]
> Voordat u begint, houd er rekening mee dat na een failover naar Azure fysieke servers kunnen niet worden niet terug naar de lokale fysieke machines. U kunt alleen failback naar VMware-machines. 


### <a name="set-up-an-azure-account"></a>Een Azure-account instellen

Ophalen van een Microsoft [Azure-account](http://azure.microsoft.com/).

- U kunt beginnen met een [gratis proefversie](https://azure.microsoft.com/pricing/free-trial/).
- Meer informatie over [prijzen voor Site Recovery](site-recovery-faq.md#pricing), en [prijsinformatie](https://azure.microsoft.com/pricing/details/site-recovery/).
- Weten wie [regio's worden ondersteund](https://azure.microsoft.com/pricing/details/site-recovery/) voor siteherstel.

### <a name="verify-azure-account-permissions"></a>Controleer de machtigingen van de Azure-account

Zorg ervoor dat uw Azure-account is gemachtigd voor de replicatie van machines naar Azure.

- Controleer de [machtigingen](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines) moet u virtuele machines repliceren naar Azure.
- Controleer en wijzig [toegangsgroepen op basis van](../role-based-access-control/role-assignments-portal.md) machtigingen. 



### <a name="set-up-an-azure-network"></a>Een Azure-netwerk instellen

Instellen van een [Azure-netwerk](../virtual-network/quick-create-portal.md).

- Virtuele machines in Azure worden geplaatst in dit netwerk wanneer ze zijn gemaakt na een failover.
- Het netwerk moet zich in dezelfde regio bevinden als de Recovery Services-kluis


## <a name="set-up-an-azure-storage-account"></a>Een Azure-opslagaccount instellen

Instellen van een [Azure storage-account](../storage/common/storage-create-storage-account.md#create-a-storage-account).

- Lokale-machines van de site Recovery worden gerepliceerd naar Azure-opslag. Virtuele machines in Azure worden gemaakt van de opslag nadat de failover plaatsvindt.
- Het opslagaccount moet zich in dezelfde regio bevinden als de Recovery Services-kluis.
- Het opslagaccount kan worden standaard of [premium](../virtual-machines/windows/premium-storage.md).
- Als u een premium-account hebt ingesteld, moet u ook een extra standaardaccount voor logboekgegevens.



### <a name="prepare-an-account-for-mobility-service-installation"></a>Een account voorbereiden voor installatie van de Mobility-service

De Mobility-service moet worden geïnstalleerd op elke server die u wilt repliceren. Site Recovery wordt deze service automatisch geïnstalleerd wanneer u replicatie voor de server inschakelen. Voor het automatisch wordt geïnstalleerd, moet u een account die door Site Recovery wordt gebruikt voor toegang tot de server voorbereiden.

- U kunt een domein of lokale account gebruiken
- Voor Windows virtuele machines, als u niet een domeinaccount, Schakel externe gebruiker toegangsbeheer op de lokale computer. Om dit te doen, in het register onder **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**, de DWORD-vermelding toevoegen **LocalAccountTokenFilterPolicy**, met een waarde van 1.
- Als u wilt de registervermelding voor het uitschakelen van de instelling van een CLI toevoegen, typt u:       ``REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1.``
- Voor Linux moet het account hoofdmap op de bronserver Linux zijn.


## <a name="create-a-vault"></a>Een kluis maken

[!INCLUDE [site-recovery-create-vault](../../includes/site-recovery-create-vault.md)]

## <a name="select-a-protection-goal"></a>Selecteer een beveiligingsdoel

Selecteer wat voor replicatie en om deze aan te repliceren.

1. Klik op **Recovery Services-kluizen** > kluis.
2. Klik in het resourcemenu op **Site Recovery** > **Infra structuur voorbereiden** > **Beveiligingsdoel**.
3. In **beveiligingsdoel**, selecteer **naar Azure** > **niet gevirtualiseerde/andere**.

## <a name="set-up-the-source-environment"></a>De bronomgeving instellen

Instellen van de configuratieserver, registreert u dit in de kluis en VM's detecteren.

1. Klik op **Site Recovery** > **infrastructuur voorbereiden** > **bron**.
2. Als u een configuratieserver geen hebt, klikt u op **+ configuratieserver**.
3. In **Server toevoegen**, controleert u of **configuratieserver** wordt weergegeven in **servertype**.
4. Download het installatiebestand van de Site Recovery Unified Setup.
5. Download de registratiesleutel voor de kluis. U moet dit wanneer u Unified Setup uitvoert. De sleutel blijft vijf dagen na het genereren ervan geldig.

   ![Bron instellen](./media/physical-azure-disaster-recovery/source-environment.png)


### <a name="register-the-configuration-server-in-the-vault"></a>De configuratieserver in de kluis registreren

Doe het volgende voordat u begint: 

- Op de servercomputer van de configuratie, zorg ervoor dat de systeemklok is gesynchroniseerd met een [tijdserver](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/get-started/windows-time-service/windows-time-service). Deze moet overeenkomen. Als het op de voorgrond 15 minuten of achter de installatie mislukken.
- Zorg ervoor dat de computer toegang heeft tot deze URL's:       [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]

- IP-adressen gebaseerde firewallregels moeten communicatie met Azure toestaan.
- Sta de [IP-adresbereiken voor Azure Datacenter](https://www.microsoft.com/download/confirmation.aspx?id=41653) en de HTTPS-poort (443) toe.
- Sta de IP-adresbereiken voor de Azure-regio van uw abonnement en voor de regio VS - west toe (deze worden gebruikt voor toegangs- en identiteitsbeheer).

Voer de installatie Unified als een lokale beheerder voor het installeren van de configuratieserver. De processerver en de hoofddoelserver worden ook standaard geïnstalleerd op de configuratieserver.

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

Nadat de registratie is voltooid, de configuratieserver wordt weergegeven op de **instellingen** > **Servers** pagina in de kluis.


## <a name="set-up-the-target-environment"></a>De doelomgeving instellen

Selecteer en controleer doelbronnen.

1. Klik op **Infrastructuur voorbereiden** > **Doel** en selecteer het Azure-abonnement dat u wilt gebruiken.
2. Geef het doel-implementatiemodel.
3. Site Recovery controleert of u een of meer compatibele Azure-opslagaccounts en -netwerken hebt.

   ![Doel](./media/physical-azure-disaster-recovery/network-storage.png)


## <a name="create-a-replication-policy"></a>Een replicatiebeleid maken

1. Klik op om een nieuw beleid voor wachtwoordreplicatie **Site Recovery-infrastructuur** > **replicatiebeleid** > **+ replicatiebeleid**.
2. In **maken van beleid voor wachtwoordreplicatie**, een beleidsnaam opgeven.
3. In **RPO drempelwaarde**, de objective (RPO) herstelpuntlimiet opgeven. Deze waarde geeft aan hoe vaak gegevens herstelpunten worden gemaakt. Wanneer de continue replicatie deze limiet overschrijdt, wordt er een waarschuwing gegenereerd.
4. In **herstel bewaarperiode**, opgeven hoe lang (in uren) bewaard voor elk herstelpunt. Gerepliceerde VM’s kunnen worden hersteld naar een willekeurig punt in een tijdvenster. Maximaal 24 uur bewaren wordt ondersteund voor computers die zijn gerepliceerd naar premium-opslag en 72 uur voor standard-opslag.
5. In **App-consistente momentopname frequentie**, opgeven hoe vaak (in minuten) herstelpunten met toepassingsconsistente momentopnamen worden gemaakt. Klik op **OK** om het beleid te maken.

    ![Beleid voor replicatie](./media/physical-azure-disaster-recovery/replication-policy.png)


Het beleid wordt automatisch gekoppeld aan de configuratieserver. Standaard wordt automatisch een bijbehorend beleid gemaakt voor failback. Bijvoorbeeld, als het replicatiebeleid **rep beleid** vervolgens een failback-beleid **rep-beleid-failback** wordt gemaakt. Dit beleid wordt pas gebruikt als u een failback initieert vanuit Azure.

## <a name="enable-replication"></a>Replicatie inschakelen

Replicatie inschakelen voor elke server.

- Site Recovery installeert de Mobility-service wanneer replicatie is ingeschakeld.
- Wanneer u replicatie voor een server inschakelt, kan het 15 minuten duren of langer voordat wijzigingen van kracht en worden weergegeven in de portal.

1. Klik op **Toepassing repliceren** > **Bron**.
2. Selecteer in **Bron** de configuratieserver.
3. In **type Machine**, selecteer **fysieke machines**.
4. Selecteer de processerver (configuratieserver). Klik vervolgens op **OK**.
5. In **doel**, selecteer het abonnement en de resourcegroep waarin u wilt maken van de Azure VM's na een failover. Kies het implementatiemodel dat u wilt gebruiken in Azure (klassiek of de resource management).
6. Selecteer het Azure-opslagaccount dat u wilt gebruiken voor het repliceren van gegevens. 
7. Selecteer het Azure-netwerk en -subnet waarmee virtuele Azure-machines verbinding maken wanneer ze na een failover worden gemaakt.
8. Selecteer **Nu configureren voor geselecteerde machines** om de netwerkinstelling toe te passen op alle machines die u voor beveiliging selecteert. Selecteer **Later configureren** om per machine een Azure-netwerk te selecteren. 
9. In **fysieke Machines**, en klik op **+ fysieke machine**. Geef de naam en IP-adres. Selecteer het besturingssysteem van de computer die u wilt repliceren. Het duurt enkele minuten voor de servers worden gedetecteerd en wordt vermeld. 
10. In **eigenschappen** > **eigenschappen configureren**, selecteer het account dat wordt gebruikt door de processerver voor het installeren van de Mobility-service automatisch op de machine.
11. Controleer of het juiste replicatiebeleid is geselecteerd in **Replicatie-instellingen** > **Replicatie-instellingen configureren**. 
12. Klik op **Replicatie inschakelen**. U kunt de voortgang van de taak **Beveiliging inschakelen** volgen via **Instellingen** > **Taken** > **Site Recovery-taken**. Nadat de taak **Beveiliging voltooien** is uitgevoerd, is de machine klaar voor een mogelijke failover.


Om te bewaken servers die u toevoegt, kunt u het tijdstip van laatste gedetecteerde voor ze op in controleren **configuratieservers** > **laatste Contact op**. Als u wilt toevoegen machines zonder te wachten op een tijdstip geplande detectie, markeer de configuratieserver (Klik niet op deze), en klik op **vernieuwen**.

## <a name="next-steps"></a>Volgende stappen

[Uitvoeren van een herstel na noodgevallen detailanalyse](tutorial-dr-drill-azure.md).
