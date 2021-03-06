---
title: Azure Service Fabric integreren met API Management | Microsoft Docs
description: In deze zelfstudie leert u hoe u snel aan de slag kunt met Azure API Management en Service Fabric.
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: 
ms.assetid: 
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 3/9/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 430e813b89f3e0004c517ef77f1028e00ebe5404
ms.sourcegitcommit: a0be2dc237d30b7f79914e8adfb85299571374ec
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/12/2018
---
# <a name="tutorial-deploy-api-management-with-service-fabric"></a>Zelfstudie: API Management implementeren met Service Fabric
Deze zelfstudie is deel vier een serie.  De implementatie van Azure API Management met Service Fabric is een geavanceerd scenario.  API Management is handig als u API's met een geavanceerde set regels voor doorsturen moet publiceren voor uw Service Fabric-services in de back-end. Cloudtoepassingen hebben meestal een gateway in de front-end nodig om een centraal ingangspunt te bieden voor gebruikers, apparaten of andere toepassingen. In Service Fabric kan een gateway elke stateless service zijn die is ontworpen voor inkomend verkeer, zoals een ASP.NET Core-toepassing, Event Hubs, IoT-Hub of Azure API Management. 

Deze zelfstudie laat zien hoe u [Azure API Management](../api-management/api-management-key-concepts.md) instelt met Service Fabric om verkeer om te leiden naar een service in de back-end van Service Fabric.  Aan het einde van de zelfstudie hebt u API Management geïmplementeerd in een VNET en een API-bewerking geconfigureerd voor het verzenden van verkeer naar -stateless services in de back-end. Zie het [overzichtsartikel](service-fabric-api-management-overview.md) voor meer informatie over Azure API Management-scenario's met Service Fabric.

In deze zelfstudie leert u het volgende:

> [!div class="checklist"]
> * API Management implementeren
> * API Management configureren
> * Een API-bewerking maken
> * Een back-endbeleid configureren
> * De API aan een product toevoegen

In deze zelfstudiereeks leert u het volgende:
> [!div class="checklist"]
> * Een veilig [Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md) of [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md) maken in Azure met behulp van een sjabloon
> * [Een cluster in- of uitschalen](service-fabric-tutorial-scale-cluster.md)
> * [De runtime van een cluster upgraden](service-fabric-tutorial-upgrade-cluster.md)
> * API Management implementeren met Service Fabric

## <a name="prerequisites"></a>Vereisten
Voor u met deze zelfstudie begint:
- Als u nog geen abonnement op Azure hebt, maak dan een [gratis account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Installeer de [Azure Powershell-module, versie 4.1 of hoger](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) of [Azure CLI 2.0](/cli/azure/install-azure-cli).
- Maak een veilig [Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md) of [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md) in Azure
- Als u een Windows-cluster implementeert, richt u een Windows-ontwikkelomgeving in. Installeer [Visual Studio 2017](http://www.visualstudio.com) en de workloads voor **Azure-ontwikkeling**, **ASP.NET-ontwikkeling en webontwikkeling** en **.NET Core platformoverschrijdende ontwikkeling**.  Richt vervolgens een [.NET-ontwikkelomgeving in](service-fabric-get-started.md).
- Als u een Linux-cluster implementeert, richt u een Java-ontwikkelomgeving in voor [Linux](service-fabric-get-started-linux.md) of [Mac OS](service-fabric-get-started-mac.md).  Installeer de [Service Fabric CLI](service-fabric-cli.md). 

## <a name="network-topology"></a>Netwerktopologie
U beschikt nu over een veilig [Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md) of [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md) in Azure en kunt dus API Management gaan implementeren in het virtuele netwerk (VNET) in het subnet en in de netwerkbeveiligingsgroep die is aangewezen voor API Management. Voor deze zelfstudie is de sjabloon API Management-Resource Manager vooraf geconfigureerd voor het gebruik van de namen van het VNET, het subnet en de netwerkbeveiligingsgroep die u hebt ingesteld in de vorige zelfstudie over het [opzetten van een Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md) of [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md). Met deze zelfstudie wordt de volgende topologie geïmplementeerd naar Azure, waarbij API Management en Service Fabric zich bevinden in subnetten van hetzelfde virtuele netwerk:

 ![Afbeelding van topologie][sf-apim-topology-overview]

## <a name="sign-in-to-azure-and-select-your-subscription"></a>Aanmelden bij Azure en uw abonnement selecteren
Meld u aan bij uw Azure-account en selecteer uw abonnement voordat u Azure-opdrachten gaat uitvoeren.

```powershell
Login-AzureRmAccount
Get-AzureRmSubscription
Set-AzureRmContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="deploy-a-service-fabric-back-end-service"></a>Een service implementeren in de back-end van Service Fabric

Voordat u API Management configureert voor het routeren van verkeer naar een service in de back-end van Service Fabric moet u eerst een actieve service maken die aanvragen kan accepteren.  Als u eerder een [Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md) hebt gemaakt, implementeert u een .NET-service voor Service Fabric.  Als u eerder een [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md) hebt gemaakt, implementeert u een service van Java voor Service Fabric.

### <a name="deploy-a-net-service-fabric-service"></a>Een .NET-service voor Service Fabric implementeren

Voor deze zelfstudie maakt u een eenvoudige stateless Reliable Service van ASP.NET Core met behulp van de standaardsjabloon voor web-API's. Hiermee maakt u een HTTP-eindpunt voor uw service, die u beschikbaar maakt via Azure API Management.

Start Visual Studio als beheerder en maak een ASP.NET Core-service:

 1. Selecteer in Visual Studio File -> New Project.
 2. Selecteer de sjabloon Service Fabric Application onder Cloud en geef deze de naam **'ApiApplication'**.
 3. Selecteer de sjabloon voor een stateless ASP.NET Core-service en geef het project de naam **'WebApiService'**.
 4. Selecteer de sjabloon voor een project van Web API ASP.NET Core 2.0.
 5. Als het project is gemaakt, opent u `PackageRoot\ServiceManifest.xml` en verwijdert u het `Port` kenmerk uit de configuratie van het eindpunt voor de resource:
 
    ```xml
    <Resources>
      <Endpoints>
        <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" />
      </Endpoints>
    </Resources>
    ```

    Door het verwijderen van de poort kan Service Fabric dynamisch een poort opgeven uit het bereik van toepassingspoorten, dat is geopend via de netwerkbeveiligingsgroep in de sjabloon van Resource Manager voor het cluster. Hierdoor kan er vanuit API Management verkeer plaatsvinden naar de poort.
 
 6. Druk in Visual Studio op F5 om te controleren of de web-API lokaal beschikbaar is. 

    Open Service Fabric Explorer en zoom in op een specifiek exemplaar van de ASP.NET Core-service om te zien op welk basisadres de service luistert. Voeg `/api/values` toe aan het basisadres en open dit adres in een browser om zo de Get-methode aan te roepen voor de ValuesController in de sjabloon voor de web-API. Het resultaat van de aanroep is de standaardreactie die door de sjabloon wordt verstuurd, te weten een JSON-matrix die twee tekenreeksen bevat:

    ```json
    ["value1", "value2"]`
    ```

    Dit is het eindpunt dat u via API Management in Azure beschikbaar maakt.

 7. Ten slotte implementeer u de toepassing in het cluster in Azure. Klik in Visual Studio met de rechtermuisknop op het toepassingsproject en selecteer **Publish**. Geef het eindpunt van het cluster op (bijvoorbeeld `mycluster.southcentralus.cloudapp.azure.com:19000`) om de toepassing te implementeren in uw Service Fabric-cluster in Azure.

Als het goed is, wordt er nu een stateless ASP.NET Core-service met de naam `fabric:/ApiApplication/WebApiService` uitgevoerd in het Service Fabric-cluster in Azure.

### <a name="create-a-java-service-fabric-service"></a>Een Java-service maken voor Service Fabric
Voor deze zelfstudie gaat u een eenvoudige webserver implementeren, die berichten via een echo-opdracht terugstuurt naar de gebruiker. De voorbeeldtoepassing voor een echo-server bevat een HTTP-eindpunt voor uw service, die u beschikbaar maakt via Azure API Management.

1. Kloon de getting started-voorbeelden voor Java.

   ```bash
   git clone https://github.com/Azure-Samples/service-fabric-java-getting-started.git
   cd service-fabric-java-getting-started/reliable-services-actor-sample
   ```

2. Bewerk *Services/EchoServer/EchoServer1.0/EchoServerApplication/EchoServerPkg/ServiceManifest.xml*. Pas het eindpunt aan zodat de service op poort 8081 luistert.

   ```xml
   <Endpoint Name="WebEndpoint" Protocol="http" Port="8081" />
   ```

3. Sla *ServiceManifest.xml* op en bouw vervolgens de toepassing EchoServer1.0.

   ```bash
   cd Services/EchoServer/EchoServer1.0/
   gradle
   ```

4. Implementeer de toepassing in het cluster.

   ```bash
   cd Scripts
   sfctl cluster select --endpoint https://mycluster.southcentralus.cloudapp.azure.com:19080 --pem <full_path_to_pem_on_dev_machine> --no-verify
   ./install.sh
   ```

   Als het goed is, wordt er nu een stateless Java-service met de naam `fabric:/EchoServerApplication/EchoServerService` uitgevoerd in het Service Fabric-cluster in Azure.

5. Open een browser en typ http://mycluster.southcentralus.cloudapp.azure.com:8081/getMessage. Als het goed is, wordt de tekst "[version 1.0]Hello World!!!" weergegeven.

## <a name="download-and-understand-the-resource-manager-templates"></a>Resource Manager-sjablonen downloaden en begrijpen
Download de volgende Resource Manager-sjablonen en parameterbestanden en sla ze op:
 
- [network-apim.json][network-arm]
- [network-apim.parameters.json][network-parameters-arm]
- [apim.json][apim-arm]
- [apim.parameters.json][apim-parameters-arm]

Met de sjabloon *netwerk apim.json* implementeert u een nieuw subnet en een nieuwe netwerkbeveiligingsgroep in het virtuele netwerk, waarop het Service Fabric-cluster wordt geïmplementeerd.

In de volgende secties worden de resources beschreven die worden gedefinieerd met de sjabloon *apim.json*. Voor meer informatie kunt u de koppelingen naar de naslagdocumentatie voor sjablonen volgen die aan elke sectie zijn toegevoegd. De configureerbare parameters die zijn gedefinieerde in het parametersbestand *apim.parameters.json* worden verderop in dit artikel besproken.

### <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service
[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service) beschrijft het exemplaar van de API Management-service: naam, SKU of laag, locatie voor resourcegroep, informatie over de uitgever en het virtuele netwerk.

### <a name="microsoftapimanagementservicecertificates"></a>Microsoft.ApiManagement/service/certificates
[Microsoft.ApiManagement/service/certificates](/azure/templates/microsoft.apimanagement/service/certificates) zorgt voor de configuratie van de beveiliging van API Management. API Management moet voor de detectie van services worden geverifieerd bij uw Service Fabric-cluster. Dit gebeurt met behulp van een clientcertificaat dat toegang tot het cluster heeft. In deze zelfstudie wordt het certificaat gebruikt dat u eerder hebt opgegeven bij het maken van het [Windows-cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md#createvaultandcert_anchor) of [Linux-cluster](service-fabric-tutorial-create-vnet-and-linux-cluster.md#createvaultandcert_anchor). Dit certificaat biedt namelijk standaard toegang tot uw cluster. 

In deze zelfstudie wordt hetzelfde certificaat gebruikt voor clientverificatie en voor de beveiliging tussen knooppunten van een cluster. U kunt een afzonderlijk clientcertificaat gebruiken als u een certificaat hebt geconfigureerd voor toegang tot uw Service Fabric-cluster. Geef de waarden voor **name**, **password** en **data** (met base-64 gecodeerde tekenreeks) op uit het bestand met de persoonlijke sleutel (.pfx) van het clustercertificaat dat u hebt opgegeven tijdens het maken van uw Service Fabric-cluster.

### <a name="microsoftapimanagementservicebackends"></a>Microsoft.ApiManagement/service/backends
[Microsoft.ApiManagement/service/backends](/azure/templates/microsoft.apimanagement/service/backends) beschrijft de back-endservice waarnaar verkeer wordt doorgestuurd. 

Voor back-ends van Service Fabric is het Service Fabric-cluster de back-end in plaats van een specifieke Service Fabric-service. Hierdoor kan met één beleid verkeer worden omgeleid naar meer dan één service in het cluster. Het veld **url** hier is een volledig gekwalificeerde servicenaam van een service in het cluster waarnaar alle aanvragen standaard worden doorgestuurd als er geen servicenaam is opgegeven in een back-endbeleid. U kunt een niet-bestaande naam gebruiken voor de service, zoals 'fabric:/fake/service' als u geen behoefte hebt aan een fallback-service. **resourceId** verwijst naar het eindpunt voor clusterbeheer.  **clientCertificateThumbprint** en **serverCertificateThumbprints** zijn de certificaten die worden gebruikt voor verificatie met het cluster.

### <a name="microsoftapimanagementserviceproducts"></a>Microsoft.ApiManagement/service/products
Met [Microsoft.ApiManagement/service/products](/azure/templates/microsoft.apimanagement/service/products) wordt een product gemaakt. In Azure API Management bevat een product een of meer API's, evenals een gebruiksquotum en de gebruiksvoorwaarden. Zodra een product is gepubliceerd, kunnen ontwikkelaars zich abonneren op het product en de API's van het product gaan gebruiken. 

Geef beschrijvende waarden op voor het product bij **displayName** en **description**. Voor deze zelfstudie is weliswaar een abonnement vereist, maar dit abonnement hoeft niet te worden goedgekeurd door een beheerder.  De **state** van dit product is 'published' en het product is dus zichtbaar voor abonnees. 

### <a name="microsoftapimanagementserviceapis"></a>Microsoft.ApiManagement/service/apis
Met [Microsoft.ApiManagement/service/apis](/azure/templates/microsoft.apimanagement/service/apis) wordt een API gemaakt. Een API in API Management vertegenwoordigt een reeks bewerkingen die kunnen worden aangeroepen door clienttoepassingen. Nadat de bewerkingen zijn toegevoegd, wordt de API toegevoegd aan een product en is deze klaar voor publicatie. Als een API is gepubliceerd, kunnen ontwikkelaars zich abonneren op de API en deze gebruiken.

- **displayName** kan elke naam zijn voor uw API. Voor deze zelfstudie gebruiken we 'Service Fabric-App'.
- **name** is een unieke en beschrijvende naam voor de API, zoals 'service-fabric-app'. Deze naam wordt weergegeven in de portals ontwikkelaars en de uitgever. 
- **serviceUrl** verwijst naar de HTTP-service die de API implementeert. API Management stuurt aanvragen door naar dit adres. De waarde van deze URL wordt niet gebruikt voor Service Fabric-back-ends. U kunt hier elke waarde invoeren. Gebruik voor deze zelfstudie bijvoorbeeld 'http://servicefabric'. 
- De waarde voor **path** wordt toegevoegd aan de basis-URL voor de API Management-service. De basis-URL is gemeenschappelijk voor alle API's die worden gehost door een exemplaar van API Management-service. In API Management worden API's herkend aan hun achtervoegsel en daarom moet het achtervoegsel uniek zijn voor elke API voor een bepaalde uitgever. 
- **protocols** bepaalt welke protocollen kunnen worden gebruikt om toegang te krijgen tot de API. Voor deze zelfstudie gebruiken en **http** en **https**.
- **path** is een achtervoegsel voor de API. Gebruik voor deze zelfstudie 'myapp'.

### <a name="microsoftapimanagementserviceapisoperations"></a>Microsoft.ApiManagement/service/apis/operations
[Microsoft.ApiManagement/service/apis/operations](/azure/templates/microsoft.apimanagement/service/apis/operations) Voordat een API in API Management kan worden gebruikt, moeten er bewerkingen worden toegevoegd aan de API.  Externe clients gebruiken een bewerking om te communiceren met de stateless service van ASP.NET Core staatloze die wordt uitgevoerd in het Service Fabric-cluster.

Geef deze waarden op als u een API-bewerking voor de front-end wilt toevoegen:

- **displayName** en **description** beschrijven de bewerking. Voor deze zelfstudie gebruiken we 'values'.
- **method** is het HTTP-woord.  Voor deze zelfstudie gebruiken we **GET**.
- **urlTemplate** wordt toegevoegd aan de basis-URL van de API en identificeert één HTTP-bewerking.  Gebruik voor deze zelfstudie `/api/values` als u de .NET-back-endservice hebt toegevoegd of `getMessage` als u de Java-back-endservice hebt toegevoegd.  De standaardinstelling is dat het URL-pad dat hier wordt opgegeven, het URL-pad is dat naar de service van Service Fabric in de back-end wordt verzonden. Als u hier het URL-pad opgeeft dat ook door de service wordt gebruikt, zoals '/api/values', werkt de bewerking zonder verdere aanpassingen. U kunt hier ook een URL-pad opgeven dat verschilt van het URL-pad dat wordt gebruikt door de service van Service Fabric in de back-end. In dat geval moet u later ook een opdracht voor wijziging van het pad opgeven in het beleid voor de bewerking.

### <a name="microsoftapimanagementserviceapispolicies"></a>Microsoft.ApiManagement/service/apis/policies
Met [Microsoft.ApiManagement/service/apis/policies](/azure/templates/microsoft.apimanagement/service/apis/policies) wordt een back-end-beleid gemaakt, waarmee alles met elkaar wordt verbonden. Dit is de plek waar u de service van Service Fabric in de back-end configureert waarnaar aanvragen worden doorgestuurd. U kunt dit beleid toepassen op elke API-bewerking.  Zie het [beleidsoverzicht](/azure/api-management/api-management-howto-policies) voor meer informatie. 

De [back-endconfiguratie voor Service Fabric](/azure/api-management/api-management-transformation-policies#SetBackendService) biedt de volgende onderdelen voor het doorsturen van aanvragen: 
 - Selectie van service-exemplaar door de naam van een exemplaar van een Service Fabric-service op te geven. Deze naam kan programmatisch worden vastgelegd (bijvoorbeeld `"fabric:/myapp/myservice"`) of worden gegenereerd vanuit de HTTP-aanvraag (bijvoorbeeld `"fabric:/myapp/users/" + context.Request.MatchedParameters["name"]`).
 - Partitie-omzetting door het genereren van een partitiesleutel met behulp van een partitieschema van Service Fabric.
 - Replicaselectie voor stateful services.
 - Voorwaarden voor opnieuw uitvoeren van omzetting waarmee u de voorwaarden kunt opgeven voor het opnieuw omzetten van een servicelocatie en het opnieuw verzenden van een aanvraag.

**policyContent** bevat de XML-inhoud van het beleid, met Json-escape.  Maak voor deze zelfstudie een back-endbeleid om aanvragen rechtstreeks door te sturen naar de stateless service van .NET of Java die eerder is geïmplementeerd. Voeg een beleid `set-backend-service` onder inbound policies.  Vervang de waarde voor *sf-service-instance-name* door `fabric:/ApiApplication/WebApiService` als u eerder de .NET back-end-service hebt geïmplementeerd of door `fabric:/EchoServerApplication/EchoServerService` als u de Java-service hebt geïmplementeerd.  *backend-id* verwijst naar een resource in de back-end, in dit geval de resource `Microsoft.ApiManagement/service/backends` die is gedefinieerd in de sjabloon *apim.json*. *backend-id* kan ook verwijzen naar een andere back-endresource die is gemaakt met behulp van de API's van API Management. Voor deze zelfstudie stelt u *backend-id* in op de waarde van de parameter *service_fabric_backend_name*.
    
```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service 
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@((int)context.Response.StatusCode != 200)" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Raadpleeg de [documentatie over back-ends voor API Management](https://docs.microsoft.com/azure/api-management/api-management-transformation-policies#SetBackendService) voor een overzicht van alle beleidskenmerken voor Service Fabric-back-ends.

## <a name="set-parameters-and-deploy-api-management"></a>Parameters instellen en API Management implementeren
Vul in *apim.parameters.json* de volgende lege parameters in voor uw implementatie. 

|Parameter|Waarde|
|---|---|
|apimInstanceName|sf-apim|
|apimPublisherEmail|myemail@contosos.com|
|apimSku|Developer|
|serviceFabricCertificateName|sfclustertutorialgroup320171031144217|
|certificatePassword|q6D7nN%6ck@6| 
|serviceFabricCertificateThumbprint|C4C1E541AD512B8065280292A8BA6079C3F26F10 |
|serviceFabricCertificate|&lt;base-64 encoded string&gt;|
|url_path|/api/values|
|clusterHttpManagementEndpoint|https://mysfcluster.southcentralus.cloudapp.azure.com:19080|
|inbound_policy|&lt;XML-tekenreeks&gt;|

De waarden voor *certificatePassword* en *serviceFabricCertificateThumbprint* moeten overeenkomen met het clustercertificaat dat is gebruikt voor het instellen van het cluster.  

*serviceFabricCertificate* is het certificaat als een met base64 gecodeerde tekenreeks, die kan worden gegenereerd met het volgende script:

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\mycertificates\sfclustertutorialgroup220171109113527.pfx");
$b64 = [System.Convert]::ToBase64String($bytes);
[System.Io.File]::WriteAllText("C:\mycertificates\sfclustertutorialgroup220171109113527.txt", $b64);
```

Vervang bij *inbound_policy* de waarde voor *sf-service-instance-name* door `fabric:/ApiApplication/WebApiService` als u eerder de .NET back-end-service hebt geïmplementeerd of door `fabric:/EchoServerApplication/EchoServerService` als u de Java-service hebt geïmplementeerd. *backend-id* verwijst naar een resource in de back-end, in dit geval de resource `Microsoft.ApiManagement/service/backends` die is gedefinieerd in de sjabloon *apim.json*. *backend-id* kan ook verwijzen naar een andere back-endresource die is gemaakt met behulp van de API's van API Management. Voor deze zelfstudie stelt u *backend-id* in op de waarde van de parameter *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service 
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@((int)context.Response.StatusCode != 200)" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Gebruik het volgende script voor het implementeren van de Resource Manager-sjabloon en de parameterbestanden voor API Management:

```powershell
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"
$templatepath="C:\clustertemplates"

New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\network-apim.json" -TemplateParameterFile "$templatepath\network-apim.parameters.json" -Verbose

New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\apim.json" -TemplateParameterFile "$templatepath\apim.parameters.json" -Verbose
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group deployment create --name ApiMgmtNetworkDeployment --resource-group $ResourceGroupName --template-file network-apim.json --parameters @network-apim.parameters.json

az group deployment create --name ApiMgmtDeployment --resource-group $ResourceGroupName --template-file apim.json --parameters @apim.parameters.json 
```

## <a name="test-it"></a>Testen

U kunt nu proberen om rechtstreeks vanuit [Azure Portal](https://portal.azure.com) via API Management een aanvraag te verzenden naar uw back-endservice in Service Fabric.

 1. Selecteer **API** in de API Management-service.
 2. Selecteer in de **Service Fabric App**-API die u hebt gemaakt in de vorige stappen het tabblad **Test** en vervolgens de bewerking **Waarden**.
 3. Klik op de knop **Verzenden** om een testaanvraag te verzenden naar de back-endservice.  U ziet een HTTP-antwoord van deze strekking:

    ```http
    HTTP/1.1 200 OK

    Transfer-Encoding: chunked

    Content-Type: application/json; charset=utf-8

    Vary: Origin

    Ocp-Apim-Trace-Location: https://apimgmtstodhwklpry2xgkdj.blob.core.windows.net/apiinspectorcontainer/PWSQOq_FCDjGcaI1rdMn8w2-2?sv=2015-07-08&sr=b&sig=MhQhzk%2FEKzE5odlLXRjyVsgzltWGF8OkNzAKaf0B1P0%3D&se=2018-01-28T01%3A04%3A44Z&sp=r&traceId=9f8f1892121e445ea1ae4d2bc8449ce4

    Date: Sat, 27 Jan 2018 01:04:44 GMT

    
    ["value1", "value2"]
    ```

## <a name="clean-up-resources"></a>Resources opschonen

Een cluster bevat de clusterresource zelf én andere Azure-resources. De eenvoudigste manier om het cluster en alle resources te verwijderen, is om de resourcegroep te verwijderen.

Meld u aan bij Azure en selecteer de abonnements-id waarmee u het cluster wilt verwijderen.  U kunt uw abonnements-id vinden door u aan te melden bij [Azure Portal](http://portal.azure.com). Verwijder de resourcegroep en alle clusterbronnen met behulp van de cmdlet [Remove-AzureRMResourceGroup](/en-us/powershell/module/azurerm.resources/remove-azurermresourcegroup).

```powershell
$ResourceGroupName = "sfclustertutorialgroup"
Remove-AzureRmResourceGroup -Name $ResourceGroupName -Force
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="next-steps"></a>Volgende stappen
In deze zelfstudie heeft u het volgende geleerd:

> [!div class="checklist"]
> * API Management implementeren
> * API Management configureren
> * Een API-bewerking maken
> * Een back-endbeleid configureren
> * De API aan een product toevoegen

[azure-powershell]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/

[apim-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.json
[apim-parameters-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.parameters.json

[network-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.json
[network-parameters-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.json

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png
