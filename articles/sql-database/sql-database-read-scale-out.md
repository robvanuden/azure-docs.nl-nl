---
title: Azure SQL Database - query's op replica's lees | Microsoft Docs
description: De Azure SQL Database biedt de mogelijkheid om te laden saldo alleen-lezen werkbelastingen met behulp van de capaciteit van de replica van een alleen-lezen - aangeroepen lezen Scale-Out.
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: article
ms.date: 04/04/2018
ms.author: sashan
ms.openlocfilehash: 0eda9012e6b6c7207d200a6e550b6bc0b0b09882
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="use-read-only-replicas-to-load-balance-read-only-query-workloads-preview"></a>Gebruik alleen-lezen partities laden werklast alleen-lezen-query (preview)

**Lees Scale-Out** kunt u saldo Azure SQL Database alleen-lezen werkbelastingen met behulp van de capaciteit van de alleen-lezen-replica's te laden. 

## <a name="overview-of-read-scale-out"></a>Overzicht van uitbreidbare lezen

Elke database in de laag Premium ([aankoopmodel DTU gebaseerde](sql-database-service-tiers.md#dtu-based-purchasing-model)) of in de essentiële zakelijke laag ([aankoopmodel vCore gebaseerde](sql-database-service-tiers.md#vcore-based-purchasing-model-preview)) automatisch wordt geleverd met verschillende Always ON replica's ondersteuning voor de beschikbaarheids-SLA. Deze replica's zijn voorzien van hetzelfde prestatieniveau als de replica van de lezen-schrijven die worden gebruikt door de normale databaseverbindingen. De **lezen Scale-Out** functie kunt u saldo SQL-Database alleen-lezen werkbelastingen met behulp van de capaciteit van de replica's alleen-lezen in plaats van delen van de replica lezen-schrijven worden geladen. Op deze manier de alleen-lezen werkbelasting geïsoleerd van de belangrijkste lezen-schrijven werkbelasting en niet van invloed op de prestaties. De functie is bedoeld voor de toepassingen die logisch zijn alleen-lezen werkbelastingen, zoals analytics, van elkaar gescheiden en daarom kunnen krijgen tot prestatievoordelen met behulp van deze extra capaciteit zonder extra kosten.

Als u wilt de lezen Scale-Out-functie met een bepaalde database gebruiken, moet u expliciet deze inschakelen bij het maken van de database of later door het wijzigen van de configuratie met behulp van PowerShell door aan te roepen de [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) of de [ Nieuwe AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) cmdlets of via de REST-API van Azure Resource Manager met behulp van de [Databases - maken of bijwerken](/rest/api/sql/databases/createorupdate) methode. 

Nadat lezen Scale-Out is ingeschakeld voor een database, verbinding maken met database toepassingen wordt omgeleid naar de alleen-lezen-replica of naar een alleen-lezen replica van die database volgens de `ApplicationIntent` eigenschap geconfigureerd in de toepassing verbindingstekenreeks. Voor informatie over de `ApplicationIntent` eigenschap, Zie [Toepassingsintentie geven](https://docs.microsoft.com/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#specifying-application-intent).

> [!NOTE]
> Tijdens de preview, worden Query Data Store en Extended Events niet ondersteund op de replica's alleen-lezen.

## <a name="data-consistency"></a>Gegevensconsistentie

Een van de voordelen van AlwaysON is dat de replica's zich altijd in de transactioneel consistent wilt maken, maar op verschillende tijdstippen in de tijd kan er een aantal kleine latentie tussen verschillende replica's. Niveau van de sessie consistentie biedt ondersteuning voor lezen Scale-Out. Het betekent als de alleen-lezen-sessie opnieuw verbinding maakt nadat een de verbindingsfout is veroorzaakt door replica ontbreken, deze kan worden omgeleid naar een replica die geen 100% up-to-date met de replica alleen-lezen. Op dezelfde manier als een toepassing schrijft gegevens via een alleen-lezen-sessie en leest onmiddellijk met behulp van een alleen-lezen-sessie, is het mogelijk de nieuwste updates zijn niet onmiddellijk zichtbaar. Dit is omdat de transactie log opnieuw op de replica's asynchroon is.

> [!NOTE]
> Replicatielatentie binnen de regio laag zijn en deze situatie zeldzame is.


## <a name="connecting-to-a-read-only-replica"></a>Verbinding maken met de replica van een alleen-lezen

Wanneer u Scale-Out van lezen voor een database inschakelt, de `ApplicationIntent` optie in de verbindingsreeks die is opgegeven door de client bepaalt of de verbinding wordt doorgestuurd naar de schrijven replica of naar een replica alleen-lezen. In het bijzonder als het `ApplicationIntent` waarde is `ReadWrite` (de standaardwaarde), de verbinding wordt omgeleid naar de databasereplica lezen-schrijven. Dit is gelijk aan bestaande gedrag. Als de `ApplicationIntent` waarde is `ReadOnly`, de verbinding wordt doorgestuurd naar een leesbare replica.

De volgende verbindingsreeks maakt de client verbinding met een alleen-lezen replica (de items in de punthaken vervangen door de juiste waarden voor uw omgeving en de punthaken weghalen):

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

Een van de volgende verbindingsreeksen de client is verbonden met een alleen-lezen-replica (de items in de punthaken vervangen door de juiste waarden voor uw omgeving en de punthaken weghalen):

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadWrite;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;

Server=tcp:<server>.database.windows.net;Database=<mydatabase>;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

## <a name="enable-and-disable-read-scale-out-using-azure-powershell"></a>Inschakelen en uitschakelen lezen Scale-Out Azure PowerShell gebruiken

Lees Scale-Out in Azure PowerShell beheren vereist dat de 2016 December release van Azure PowerShell of hoger. Zie voor de nieuwste versie van de PowerShell [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps).

In- of lezen scale-out in Azure PowerShell uitschakelen door het aanroepen van de [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) cmdlet en doorgegeven in de gewenste waarde – `Enabled` of `Disabled` --voor de `-ReadScale` parameter. U kunt ook de [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) cmdlet voor het maken van een nieuwe database met lezen scale-out is ingeschakeld.

Bijvoorbeeld, om in te schakelen dat is gelezen scale-out voor een bestaande database (de items in de punthaken vervangen door de juiste waarden voor uw omgeving en de punthaken weghalen):

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled
```

Lees scale-out voor een bestaande database (de items in de punthaken vervangen door de juiste waarden voor uw omgeving en de punthaken weghalen) uitschakelen:

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Disabled
```

Een nieuwe database maken met lezen scale-out ingeschakeld (de items in de punthaken vervangen door de juiste waarden voor uw omgeving en de punthaken weghalen):

```powershell
New-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled -Edition Premium
```

## <a name="enabling-and-disabling-read-scale-out-using-the-azure-sql-database-rest-api"></a>Inschakelen en uitschakelen van lezen Scale-Out met de REST-API van Azure SQL Database

Maken van een database met lezen scale-out is ingeschakeld, of als u wilt in- of uitschakelen lezen scale-out voor een bestaande database, maken of bijwerken van de overeenkomende entiteit van de database met de `readScale` eigenschap ingesteld op `Enabled` of `Disabled` als in het onderstaande voorbeeld de aanvraag.

```rest
Method: PUT
URL: https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{GroupName}/providers/Microsoft.Sql/servers/{ServerName}/databases/{DatabaseName}?api-version= 2014-04-01-preview
Body:
{
   "properties":
   {
      "readScale":"Enabled"
   }
} 
```

Zie voor meer informatie, [Databases - maken of bijwerken](/rest/api/sql/databases/createorupdate).

## <a name="next-steps"></a>Volgende stappen

- Zie voor meer informatie over het gebruik van PowerShell lezen scale-out instellen de [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) of de [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) cmdlets.
- Zie voor meer informatie over het gebruik van de REST-API om in te stellen lezen scale-out [Databases - maken of bijwerken](/rest/api/sql/databases/createorupdate).
