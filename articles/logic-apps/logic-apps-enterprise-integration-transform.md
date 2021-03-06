---
title: Converteren van XML-gegevens met transformaties - Azure Logic Apps | Microsoft Docs
description: Maken van transformaties of mapps converteren van XML-gegevens tussen indelingen in logic apps met behulp van de Enterprise Integration-SDK
services: logic-apps
documentationcenter: .net,nodejs,java
author: msftman
manager: anneta
editor: cgronlun
ms.assetid: add01429-21bc-4bab-8b23-bc76ba7d0bde
ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/08/2016
ms.author: LADocs; padmavc
ms.openlocfilehash: fd59b6b3f51adb538e774bc5bb089880ca22e97e
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/28/2018
---
# <a name="enterprise-integration-with-xml-transforms"></a>Enterprise-integratie met XML-transformaties
## <a name="overview"></a>Overzicht
De Enterprise integration transformatie-connector worden gegevens van de ene indeling geconverteerd naar een andere indeling. Bijvoorbeeld wellicht hebt u een binnenkomend bericht met de huidige datum in de notatie YearMonthDay. U kunt een transformatie opnieuw indelen van de datum in de indeling MonthDayYear.

## <a name="what-does-a-transform-do"></a>Wat is een transformatie?
Een transformatie die ook wel bekend als een kaart, bestaat uit een bron-XML-schema (invoer) en een doel-XML-schema (uitvoer). U kunt verschillende ingebouwde functies gebruiken om te bewerken of beheren van de gegevens, waaronder tekenreeksmanipulatie, voorwaardelijke toewijzingen, aritmetische expressies, datum tijd formatters en zelfs lussen constructies.

## <a name="how-to-create-a-transform"></a>Het maken van een transformatie?
U kunt een transformatie/map maken met behulp van de Visual Studio [Enterprise Integration SDK](https://aka.ms/vsmapsandschemas). Als u klaar maken en testen van de transformatie bent, moet u de transformatie toegang tot je account integratie uploaden. 

## <a name="how-to-use-a-transform"></a>Het gebruik van een transformatie
Nadat u de transformatie/kaart bij uw account integratie geüpload, kunt u het maken van een logische app. De logische app wordt uw transformaties uitgevoerd wanneer de logische app wordt geactiveerd (en er is invoer inhoud die moet worden omgezet).

**Hier volgen de stappen voor het gebruik van een transformatie**:

### <a name="prerequisites"></a>Vereisten

* Een integratie-account maken en een toewijzing aan toe te voegen  

Nu dat u hebt gezorgd voor de vereisten, is het tijd om uw logische app maken:  

1. Een logische app maken en [koppelen aan uw account integratie](../logic-apps/logic-apps-enterprise-integration-accounts.md "informatie over het koppelen van een integratie-account aan een logische app") die de kaart bevat.
2. Voeg een **aanvragen** trigger aan uw logische app  
   ![](./media/logic-apps-enterprise-integration-transforms/transform-1.png)    
3. Toevoegen de **XML-transformatie** actie selecteren **een actie toevoegen**   
   ![](./media/logic-apps-enterprise-integration-transforms/transform-2.png)   
4. Voer het woord *transformeren* in het zoekvak voor het filteren van de acties met de fout die u wilt gebruiken  
   ![](./media/logic-apps-enterprise-integration-transforms/transform-3.png)  
5. Selecteer de **XML-transformatie** actie   
6. Voeg het XML-bestand **inhoud** die u transformeren. U in de HTTP-aanvraag als ontvangt XML-gegevens kunt u de **inhoud**. Selecteer de hoofdtekst van de HTTP-aanvraag die de logische app heeft geactiveerd in dit voorbeeld.

   > [!NOTE]
   > Zorg ervoor dat de inhoud voor de **XML-transformatie** XML. Als de inhoud zich niet in XML of base64-gecodeerd, moet u een expressie waarmee de inhoud wordt verwerkt. U kunt bijvoorbeeld [functies](logic-apps-workflow-definition-language.md#functions), bijvoorbeeld ```@base64ToBinary``` voor het decoderen van inhoud of ```@xml``` voor het verwerken van de inhoud als XML.
 

7. Selecteer de naam van de **kaart** dat u gebruiken wilt voor het uitvoeren van de transformatie. De kaart moet al in uw account integratie. In een eerdere stap hebt u al gegeven uw logische apptoegang tot je account integratie met uw kaart.      
   ![](./media/logic-apps-enterprise-integration-transforms/transform-4.png) 
8. Sla uw werk  
    ![](./media/logic-apps-enterprise-integration-transforms/transform-5.png) 

Op dit moment bent u klaar instellen van de kaart. In een toepassing praktijk wilt u mogelijk de getransformeerde gegevens opslaan in een LOB-toepassing zoals SalesForce. U kunt gemakkelijk als een actie voor het verzenden van de uitvoer van de transformatie bij Salesforce. 

U kunt nu de transformatie testen door een aanvraag voor het HTTP-eindpunt.  


## <a name="features-and-use-cases"></a>Functies en gebruiksvoorbeelden
* De transformatie die wordt gemaakt in een map is eenvoudig, zoals het kopiëren van een naam en adres van het ene document naar de andere. Of u complexere transformaties met behulp van de kaart out-of-the-box-bewerkingen kunt maken.  
* Meerdere bewerkingen van de kaart of functies zijn direct beschikbaar is, met inbegrip van tekenreeksen, Datum tijdfuncties, enzovoort.  
* U kunt een kopie van de directe gegevens tussen de schema's kunt doen. In de Mapper die deel uitmaakt van de SDK is net zo eenvoudig als het tekenen van een regel die de elementen in het schema van de gegevensbron met hun collega's in het doelschema verbindt.  
* Wanneer u een toewijzing maakt, weergave u een grafische van de kaart, waarin de relaties en koppelingen die u maakt.
* Gebruik de functie Test-kaart toevoegen van een bericht van de XML-voorbeeld. U kunt met een enkele klik testen van de kaart die u hebt gemaakt en verschijnt de gegenereerde uitvoer.  
* Bestaande maps uploaden  
* Biedt ondersteuning voor de XML-indeling.

## <a name="advanced-features"></a>Geavanceerde functies

### <a name="reference-assembly-or-custom-code-from-maps"></a>Referentie-assembly of aangepaste code van maps 
De transformatie-actie maps ondersteunt ook of met een verwijzing naar externe assembly transformaties. Deze mogelijkheid kan aanroepen van aangepaste .NET-code rechtstreeks vanuit de XSLT-kaarten. Hier volgen de vereisten voor het gebruik van assembly van maps.

* De kaart en de assembly waarnaar wordt verwezen vanuit de kaart moet worden [geüpload naar het account van de integratie](./logic-apps-enterprise-integration-maps.md). 

  > [!NOTE]
  > Kaart en assembly zijn vereist om te worden geüpload in een bepaalde volgorde. Voordat u de kaart die verwijst naar de assembly uploadt, moet u de assembly uploaden.

* De kaart moet ook beschikken over deze kenmerken en een CDATA-sectie dat de aanroep van de assembly-code bevat:

    * **naam** is de aangepaste assembly-naam.
    * **naamruimte** is de naamruimte in de assembly waarin de aangepaste code.

  Dit voorbeeld ziet u een toewijzing die verwijst naar een assembly met de naam 'XslUtilitiesLib' en roept de `circumreference` methode uit de assembly.

  ````xml
  <?xml version="1.0" encoding="UTF-8"?>
  <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="urn:my-scripts">
  <msxsl:script language="C#" implements-prefix="user">
    <msxsl:assembly name="XsltHelperLib"/>
    <msxsl:using namespace="XsltHelpers"/>
    <![CDATA[public double circumference(int radius){ XsltHelper helper = new XsltHelper(); return helper.circumference(radius); }]]>
  </msxsl:script>
  <xsl:template match="data">
     <circles>
        <xsl:for-each select="circle">
            <circle>
                <xsl:copy-of select="node()"/>
                    <circumference>
                        <xsl:value-of select="user:circumference(radius)"/>
                    </circumference>
            </circle>
        </xsl:for-each>
     </circles>
    </xsl:template>
    </xsl:stylesheet>
  ````


### <a name="byte-order-mark"></a>Byte Order Mark
Het antwoord van de transformatie wordt standaard gestart met de Byte Order Mark (BOM). U kunt toegang tot deze functionaliteit alleen tijdens het werken in de weergave Code-editor. Geef om deze functionaliteit is uitgeschakeld, `disableByteOrderMark` voor de `transformOptions` eigenschap:

````json
"Transform_XML": {
    "inputs": {
        "content": "@{triggerBody()}",
        "integrationAccount": {
            "map": {
                "name": "TestMap"
            }
        },
        "transformOptions": "disableByteOrderMark"
    },
    "runAfter": {},
    "type": "Xslt"
}
````





## <a name="learn-more"></a>Meer informatie
* [Meer informatie over het Enterprise-integratiepakket](../logic-apps/logic-apps-enterprise-integration-overview.md "meer informatie over Enterprise Integration Pack")  
* [Meer informatie over maps](../logic-apps/logic-apps-enterprise-integration-maps.md "meer informatie over enterprise integration maps")  

