---
title: De Databricks Notebook-activiteit uitvoeren in Azure Data Factory op een Databricks Notebook
description: Meer informatie over hoe u de Databricks Notebook-activiteit in een Azure Data Factory kunt gebruiken voor het uitvoeren van een Databricks-notebook in het Databricks-takencluster.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/12/2018
ms.author: abnarain
ms.reviewer: douglasl
ms.openlocfilehash: edd3bbb5d47753890328ae86b15dc938dfcf7eb7
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/05/2018
---
# <a name="run-a-databricks-notebook-with-the-databricks-notebook-activity-in-azure-data-factory"></a>De Databricks Notebook-activiteit uitvoeren in Azure Data Factory op een Databricks-notebook

In deze zelfstudie gebruikt u Azure Portal om een Azure Data Factory-pijplijn te maken die een Databricks-notebook uitvoert op basis van het Databricks-takencluster. Bovendien worden tijdens de uitvoering Azure Data Factory-parameters doorgestuurd naar de Databricks-notebook.

In deze zelfstudie voert u de volgende stappen uit:

  - Een data factory maken.

  - Een pijplijn maken die gebruikmaakt van Databricks Notebook-activiteit.

  - Een pijplijnuitvoering activeren.

  - De pijplijnuitvoering controleert.

Als u nog geen abonnement op Azure hebt, maak dan een [gratis account](https://azure.microsoft.com/free/) aan voordat u begint.

## <a name="prerequisites"></a>Vereisten

  - **Azure Databricks-werkruimte**. [Maak een Databricks-werkruimte](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal) of gebruik een bestaande werkruimte. U maakt een Python-notebook in uw Azure Databricks-werkruimte. Vervolgens voert u de notebook uit en geeft u er parameters aan door met behulp van Azure Data Factory.

## <a name="create-a-data-factory"></a>Een gegevensfactory maken

1.  Start de webbrowser **Microsoft Edge** of **Google Chrome**. Op dit moment wordt de Data Factory-gebruikersinterface alleen ondersteund in de webbrowsers Microsoft Edge en Google Chrome.

2.  Selecteer **Nieuw** in het menu links, selecteer **Gegevens en analyses** en selecteer vervolgens **Data Factory**.

    ![Een nieuwe gegevensfactory maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image1.png)

3.  Voer op de pagina **Nieuwe data factory** **ADFTutorialDataFactory** in bij **Naam**.

    De naam van de Azure-gegevensfactory moet *wereldwijd uniek* zijn. Als u de volgende fout ziet, wijzigt u de naam van de gegevensfactory. (Gebruik dan bijvoorbeeld **\<uwnaam\>ADFTutorialDataFactory**). Zie het artikel [Data factory - Naamgevingsregels](https://docs.microsoft.com/en-us/azure/data-factory/naming-rules) voor naamgevingsregels voor Data Factory-artefacten.

    ![Een naam opgeven voor de nieuwe gegevensfactory](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image2.png)

4.  Selecteer voor **Abonnement** het Azure-abonnement waarin u de gegevensfactory wilt maken.

5.  Voer een van de volgende stappen uit voor **Resourcegroep**:
    
    - Selecteer **Bestaande gebruiken** en selecteer een bestaande resourcegroep in de vervolgkeuzelijst.
    
    - Selecteer **Nieuwe maken** en voer de naam van een resourcegroep in.

    Voor sommige van de stappen in deze snelstartgids wordt ervan uitgegaan dat u voor de resourcegroep de naam **ADFTutorialResourceGroup** gebruikt. Zie [Resourcegroepen gebruiken om Azure-resources te beheren](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) voor meer informatie.

1.  Selecteer **V2 (Preview)** voor **Versie**.

2.  Selecteer voor **Locatie** een locatie voor de data factory.

    Momenteel kunt u in Data Factory V2 alleen data factory's maken in de regio's VS - oost, VS - oost 2 en West-Europa. De gegevensarchieven (zoals Azure Storage en Azure SQL Database) en berekeningen (zoals Azure HDInsight) die Data Factory gebruikt, kunnen zich in andere regio's bevinden.

3.  Selecteer **Vastmaken aan dashboard**.

4.  Selecteer **Maken**.

5.  Op het dashboard ziet u de volgende tegel met de status: **Data Factory implementeren**:

    ![](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image3.png)

6.  Zodra de bewerking is voltooid, ziet u de pagina **Data Factory**. Selecteer de tegel **Maken en controleren** om de gebruikersinterface van Data Factory te openen op een afzonderlijk tabblad.

    ![De gebruikersinterface van Data Factory starten](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image4.png)

## <a name="create-linked-services"></a>Gekoppelde services maken

In deze sectie maakt u een aan Databricks gekoppelde service. Deze gekoppelde service bevat de verbindingsgegevens voor het Databricks-cluster:

### <a name="create-an-azure-databricks-linked-service"></a>Een aan Azure Databricks gekoppelde service maken

1.  Schakel op de pagina **Aan de slag** naar het tabblad **Bewerken** in het linkerpaneel.

    ![De nieuwe gekoppelde service opslaan](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image5.png)

2.  Selecteer **Verbindingen** onder aan het venster en selecteer **+ Nieuw**.
    
    ![Een nieuwe verbinding maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image6.png)

3.  Selecteer in het venster **Nieuwe gekoppelde service** de optie **Gegevensarchief**\>**Azure Databricks** en selecteer vervolgens **Doorgaan**.
    
    ![Een aan Databricks gekoppelde service opgeven](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image7.png)

4.  Voer in het venster **Nieuwe gekoppelde service** de volgende stappen uit:
    
    1.  Voer bij **Naam** ***AzureDatabricks\_LinkedService*** in.
    
    2.  Selecteer voor **Cluster** een **Nieuw cluster**
    
    3.  Selecteer voor **Domein/regio** de regio waar uw Azure Databricks-werkruimte zich bevindt.
    
    4.  Selecteer voor **Cluster-knooppunttype** de optie **Standaard\_D3\_v2** voor deze zelfstudie.
    
    5.  Genereer de **Toegangstoken** vanuit de Azure Databricks-werkruimte. U kunt de daarvoor benodigde stappen [hier](https://docs.databricks.com/api/latest/authentication.html#generate-token) vinden.
    
    6.  Selecteer voor **Clusterversie** de optie **4.0 Bèta** (meest recente versie)
    
    7.  Bij **Aantal worker-knooppunten** voert u **2** in.
    
    8.  Selecteer **Voltooien**.

        ![Het maken van de gekoppelde service voltooien](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image8.png)

## <a name="create-a-pipeline"></a>Een pijplijn maken

1.  Selecteer de knop **+** (plusteken) en selecteer vervolgens **Pijplijn** in het menu.

    ![Knoppen voor het maken van een nieuwe pijplijn](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image9.png)

2.  Maak een **parameter** voor gebruik in de **pijplijn**. Deze parameter geeft u later door aan de Databricks Notebook-activiteit. Open in de lege pijplijn het tabblad **Parameters**, klik op **Nieuw** en noem deze '**naam**'.

    ![Een nieuwe parameter maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image10.png)

    ![De naam van de parameter maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image11.png)

3.  Vouw in de werkset **Activiteiten** **Databricks** uit. Sleep de activiteit **Notebook** vanuit de werkset **Activiteiten** naar het ontwerpoppervlak voor pijplijnen.

    ![Sleep de notebook naar het ontwerpoppervlak](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image12.png)

4.  Voer de volgende stappen uit in de eigenschappen voor het **Databricks** **Notebook**-activiteitvenster onderaan:

    a. Schakel over naar het tabblad **Instellingen**.

    b. Selecteer **myAzureDatabricks\_LinkedService** (gemaakt in de vorige procedure).

    c. Selecteer een Databricks **Notebook-pad**. We gaan een notebook maken en geven daarvoor hier het pad op. U krijgt het pad van de notebook door de volgende stappen uit te voeren.

       1. De Azure Databricks-werkruimte openen

       2. Maak een **Nieuwe map** in de werkruimte en roep deze aan als **adftutorial**.

          ![Een nieuwe map maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image13.png)

       3. [Maak een nieuwe notebook](https://docs.databricks.com/user-guide/notebooks/index.html#creating-a-notebook) (Python) met de naam **mynotebook** in de map **adftutorial****** en klik op **Maken.**

          ![Een nieuwe notebook maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image14.png)

          ![De eigenschappen van de nieuwe notebook instellen](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image15.png)

       4. In de zojuist gemaakte notebook 'mynotebook' voegt u de volgende code toe:

           ```
           # Creating widgets for leveraging parameters, and printing the parameters

           dbutils.widgets.text("input", "","")
           dbutils.widgets.get("input")
           y = getArgument("input")
           print "Param -\'input':"
           print y
           ```

           ![Widgets voor parameters maken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image16.png)

       5. In dit geval is het **Notebook-pad** **/adftutorial/mynotebook**

5.  Ga terug naar de **gebruikersinterface van Data Factory**. Ga naar het tabblad **Instellingen** onder **Notebook1-activiteit**. 
    
    a.  **Voeg een parameter toe** aan de Notebook-activiteit. U gebruikt dezelfde parameter die u eerder aan de **pijplijn** hebt toegevoegd.

       ![Een parameter toevoegen](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image17.png)

    b.  Geef de parameter de naam **input** en geef de waarde op als de expressie  **@pipeline().parameters.name**.

6.  Selecteer op de werkbalk de knop **Valideren** om de pijplijn te valideren. Selecteer de **\>\>** (pijl-rechts) om het validatievenster te sluiten.

    ![De pijplijn valideren](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image18.png)

7.  Selecteer **Alles publiceren**. De gebruikersinterface van Data Factory publiceert entiteiten (gekoppelde services en pijplijn) naar de Azure Data Factory-service.

    ![De nieuwe Data Factory-entiteiten publiceren](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image19.png)

## <a name="trigger-a-pipeline-run"></a>Een pijplijnuitvoering activeren

Selecteer op de werkbalk de optie **Activeren** en selecteer vervolgens **Nu activeren**.

![Opdracht Nu activeren selecteren](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image20.png)

Het dialoogvenster **Pijplijnuitvoering**vraagt naar de **naam**-parameter. Gebruik hier **/pad/bestandsnaam** als parameter. Klik op **Voltooien**.

![Een waarde voor de naam-parameters opgeven](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image21.png)

## <a name="monitor-the-pipeline-run"></a>De pijplijnuitvoering controleren.

1.  Ga naar het tabblad **Controleren**. Controleer of u een pijplijnuitvoering ziet. Het duurt 5 tot 8 minuten om een Databricks-taakcluster te maken, waar de notebook wordt uitgevoerd.

    ![De pijplijn bewaken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image22.png)

2.  Selecteer regelmatig **Vernieuwen** om de status van de pijplijnuitvoering te controleren.

3.  Selecteer **Uitvoering van activiteiten weergeven** in de kolom **Acties** om de uitvoering van activiteiten weer te geven die zijn gekoppeld aan de pijplijnuitvoering.

    ![Uitvoeringen van activiteit bekijken](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image23.png)

Als u wilt terugkeren naar de vorige weergave, selecteert u **Pijplijnen** bovenaan.

## <a name="verify-the-output"></a>De uitvoer controleren

Meld u aan bij de **Azure Databricks-werkruimte**, ga naar **Clusters** en de **Taak**-status wordt weergegeven als *uitvoering in behandeling, wordt uitgevoerd of beëindigd*.

![De clustertaak en de taak weergeven](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image24.png)

U kunt klikken op de **Taaknaam** om meer informatie weer te geven. Na een gelukte uitvoering kunt u de doorgegeven parameters en de uitvoer van de Python-notebook valideren.

![Uitvoeringsdetails en uitvoer weergeven](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image25.png)

## <a name="next-steps"></a>Volgende stappen

De pijplijn in dit voorbeeld activeert een Databricks Notebook-activiteit en geeft daar een parameter aan door. U hebt geleerd hoe u:

  - Een data factory maken.

  - Een pijplijn maken die gebruikmaakt van de Databricks Notebook-activiteit.

  - Een pijplijnuitvoering activeren.

  - De pijplijnuitvoering controleert.
