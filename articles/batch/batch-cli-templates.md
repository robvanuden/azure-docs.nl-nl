---
title: Azure Batch-taken end-to-end worden uitgevoerd zonder het schrijven van code (Preview) | Microsoft Docs
description: Sjabloonbestanden maken voor de Azure CLI voor het maken van de Batch-pools, jobs en taken.
services: batch
author: mscurrell
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: na
ms.topic: article
ms.workload: big-compute
ms.date: 12/18/2017
ms.author: markscu
ms.openlocfilehash: 4b4a5b9199fe648425304eaa8db0130bb1b4264d
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/05/2018
---
# <a name="use-azure-batch-cli-templates-and-file-transfer-preview"></a>Azure Batch CLI-sjablonen en -bestandsoverdracht gebruiken (preview)

Met de Azure CLI is het mogelijk batchtaken uitvoeren zonder code te schrijven.

Maken en gebruiken van sjabloonbestanden met de Azure CLI voor het maken van de Batch-pools, jobs en taken. Eenvoudig uploaden taak invoerbestanden naar het opslagaccount dat is gekoppeld met de Batch-account en downloadtaak uitvoerbestanden.

## <a name="overview"></a>Overzicht

Een uitbreiding van de Azure CLI kunt Batch moet gebruikte end-to-end door gebruikers die geen ontwikkelaars. Met alleen CLI-opdrachten, kunt u een groep maken, invoergegevens uploaden, taken en de bijbehorende taken maken en de resulterende uitvoergegevens downloaden. Er is geen aanvullende code vereist. De CLI-opdrachten rechtstreeks uit te voeren of scripts integreren.

Batch-sjablonen samenstellen op de [bestaande Batch-ondersteuning in de Azure CLI](batch-cli-get-started.md#json-files-for-resource-creation) voor JSON-bestanden bij het maken van groepen, taken, taken en andere items eigenschapswaarden opgeven. Met Batch-sjablonen zijn de volgende mogelijkheden toegevoegd via wat is er mogelijk met de JSON-bestanden:

-   Parameters kunnen worden gedefinieerd. Wanneer de sjabloon die wordt gebruikt, worden alleen de parameterwaarden die zijn opgegeven voor het maken van het item met een ander item eigenschapswaarden worden opgegeven in de hoofdtekst van de sjabloon. Een gebruiker die Batch begrijpt en de toepassingen worden uitgevoerd door de Batch kunnen sjablonen maken, pool, Jobs en taak eigenschapswaarden opgeven. Een gebruiker minder bekend bent met Batch-en/of de toepassingen hoeft alleen de waarden opgeven voor de gedefinieerde parameters.

-   Taak taak fabrieken maken een of meer taken die zijn gekoppeld aan een taak, waardoor de noodzaak van veel taakdefinities gemaakt voorkomen en taak verzending aanzienlijk te vereenvoudigen.


Taken invoergegevensbestanden gebruiken doorgaans en gegevensbestanden uitvoer wordt geproduceerd. Een opslagaccount is standaard elke Batch-account gekoppeld. Overdracht van bestanden naar en van dit opslagaccount met de CLI geen coderen en geen referenties voor de opslag.

Bijvoorbeeld: [ffmpeg](http://ffmpeg.org/) is een populaire toepassing die audio en video-bestanden worden verwerkt. Hier volgen de stappen met de Azure Batch CLI aan te roepen ffmpeg te transcoderen videobestanden bronbestanden naar verschillende oplossingen.

-   Maak een sjabloon voor de groep van toepassingen. De gebruiker die het maken van de sjabloon kent het aanroepen van de toepassing ffmpeg en de vereisten; ze geven de juiste OS, VM grootte, hoe ffmpeg is geïnstalleerd (van een toepassingspakket of met behulp van een Pakketbeheer bijvoorbeeld), en andere eigenschapswaarden-groep. Parameters worden gemaakt wanneer de sjabloon die wordt gebruikt, alleen de pool-ID en het aantal virtuele machines moeten worden opgegeven.

-   Maak een taaksjabloon. De gebruiker die de sjabloon maakt weet hoe ffmpeg moet worden aangeroepen om te transcoderen bronvideo naar een andere resolutie en geeft de opdrachtregel van de taak; ze ook weten dat er een map met de bronbestanden video met een taak vereist per bestand voor invoer.

-   Een eindgebruiker met een reeks transcoderen videobestanden maakt eerst een groep met de sjabloon toepassingen geven alleen de pool-ID en het aantal virtuele machines die zijn vereist. Ze kunnen vervolgens de bronbestanden te transcoderen uploaden. Een taak kan vervolgens worden verzonden met behulp van de taaksjabloon geven alleen de pool-ID en de locatie van de bronbestanden geüpload. De Batch-job is gemaakt met één taak per bestand voor invoer wordt gegenereerd. Ten slotte kunnen de uitvoerbestanden getranscodeerd worden gedownload.

## <a name="installation"></a>Installatie

Een Azure Batch CLI-uitbreiding voor het gebruik van de sjabloon en het bestand overdrachtmogelijkheden te installeren.

Zie voor instructies over het installeren van de Azure CLI [2.0 voor Azure CLI installeren](/cli/azure/install-azure-cli).

Zodra de Azure CLI is geïnstalleerd, installeert u de nieuwste versie van de Batch-uitbreiding met de volgende CLI-opdracht:

```azurecli
az extension add --name azure-batch-cli-extensions
```

Zie voor meer informatie over de Batch-uitbreiding [Microsoft Azure Batch CLI uitbreidingen voor Windows, Mac en Linux](https://github.com/Azure/azure-batch-cli-extensions#microsoft-azure-batch-cli-extensions-for-windows-mac-and-linux).

## <a name="templates"></a>Sjablonen

De Batch Azure CLI kunt items zoals pools, jobs en taken kunnen worden gemaakt door te geven van een JSON-bestand met de namen van eigenschappen en waarden. Bijvoorbeeld:

```azurecli
az batch pool create –-json-file AppPool.json
```

Azure Batch-sjablonen zijn vergelijkbaar met Azure Resource Manager-sjablonen, in de functionaliteit en syntaxis. Ze zijn JSON-bestanden die itemnamen en waarden bevatten, maar de volgende belangrijke concepten toevoegen:

-   **Parameters**

    -   Toestaan dat eigenschapswaarden worden opgegeven in een sectie hoofdtekst met alleen parameterwaarden hoeft te worden opgegeven wanneer de sjabloon die wordt gebruikt. De volledige definitie voor een groep kan bijvoorbeeld worden geplaatst in de hoofdtekst en slechts één parameter gedefinieerd voor groep-id; alleen een tekenreeks in de pool-ID moet daarom worden opgegeven voor het maken van een groep.
        
    -   De hoofdtekst van de sjabloon kan worden ontworpen door iemand met kennis van de Batch- en de toepassingen worden uitgevoerd door de Batch; alleen de waarden voor de auteur gedefinieerde parameters moeten worden opgegeven wanneer de sjabloon die wordt gebruikt. Een gebruiker zonder de diepgaande Batch en/of de toepassing kennis kan daarom de sjablonen gebruiken.

-   **Variabelen**

    -   Toestaan van eenvoudige of complexe parameterwaarden worden opgegeven in één locatie en gebruikt op een of meer plaatsen in de hoofdtekst van de sjabloon. Variabelen kunnen vereenvoudigen en Reduceer de grootte van de sjabloon, evenals meer bruikbaar maken door één locatie om eigenschappen te wijzigen.

-   **Een hoger niveau constructies**

    -   Sommige op een hoger niveau constructies zijn beschikbaar in de sjabloon die nog niet beschikbaar in de Batch-API's. Een taak factory kan bijvoorbeeld worden gedefinieerd in een sjabloon die wordt gemaakt van meerdere taken voor de taak met een gemeenschappelijke taakdefinitie. Deze gegevens hoeft op te code voor het dynamisch Maak meerdere JSON-bestanden, zoals één bestand per taak, evenals scriptbestanden maken voor het installeren van toepassingen via een Pakketbeheer.

    -   Op een bepaald moment mogelijk deze constructies toegevoegd aan de Batch-service en beschikbaar in de Batch-API's, UI, enzovoort.

### <a name="pool-templates"></a>Pool-sjablonen

Sjablonen van de groep van toepassingen ondersteunen de standaardsjabloon-mogelijkheden van de parameters en variabelen. Ze bieden ook ondersteuning voor de volgende een hoger niveau construct:

-   **Pakket met verwijzingen**

    -   Desgewenst kunt software wordt gekopieerd naar de knooppunten van de groep van toepassingen met behulp van pakket managers. De package manager en de pakket-ID worden opgegeven. Declareert een of meer pakketten, kunt u vermijden maken van een script dat de vereiste pakketten ophaalt, installeren van het script en het script uitgevoerd op elk knooppunt van de groep van toepassingen.

Hier volgt een voorbeeld van een sjabloon die wordt gemaakt van een pool van Linux virtuele machines met ffmpeg geïnstalleerd. Als u wilt gebruiken, alleen een tekenreeks in de pool-ID en het aantal virtuele machines in de groep op te geven:

```json
{
    "parameters": {
        "nodeCount": {
            "type": "int",
            "metadata": {
                "description": "The number of pool nodes"
            }
        },
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The pool ID "
            }
        }
    },
    "pool": {
        "type": "Microsoft.Batch/batchAccounts/pools",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('poolId')]",
            "virtualMachineConfiguration": {
                "imageReference": {
                    "publisher": "Canonical",
                    "offer": "UbuntuServer",
                    "sku": "16.04.0-LTS",
                    "version": "latest"
                },
                "nodeAgentSKUId": "batch.node.ubuntu 16.04"
            },
            "vmSize": "STANDARD_D3_V2",
            "targetDedicatedNodes": "[parameters('nodeCount')]",
            "enableAutoScale": false,
            "maxTasksPerNode": 1,
            "packageReferences": [
                {
                    "type": "aptPackage",
                    "id": "ffmpeg"
                }
            ]
        }
    }
}
```

Als de naam van het sjabloonbestand _groep ffmpeg.json_, klikt u vervolgens de sjabloon als volgt starten:

```azurecli
az batch pool create --template pool-ffmpeg.json
```

### <a name="job-templates"></a>Taaksjablonen

Taaksjablonen ondersteuning voor de standaardsjabloon-mogelijkheden van de parameters en variabelen. Ze bieden ook ondersteuning voor de volgende een hoger niveau construct:

-   **Taak factory**

    -   Meerdere taken voor een taak maakt van de definitie van één taak. Drie soorten taak factory worden ondersteund: parametrische sweep, taak per bestand, en verzameling van de taak.

Hier volgt een voorbeeld van een sjabloon die u een taak voor het transcoderen MP4 videobestanden met ffmpeg op een van twee lagere resolutie maakt. Hiermee maakt u één taak per bronbestand video:

```json
{
    "parameters": {
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch pool which runs the job"
            }
        },
        "jobId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch job"
            }
        },
        "resolution": {
            "type": "string",
            "defaultValue": "428x240",
            "allowedValues": [
                "428x240",
                "854x480"
            ],
            "metadata": {
                "description": "Target video resolution"
            }
        }
    },
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('jobId')]",
            "constraints": {
                "maxWallClockTime": "PT5H",
                "maxTaskRetryCount": 1
            },
            "poolInfo": {
                "poolId": "[parameters('poolId')]"
            },
            "taskFactory": {
                "type": "taskPerFile",
                "source": { 
                    "fileGroup": "ffmpeg-input"
                },
                "repeatTask": {
                    "commandLine": "ffmpeg -i {fileName} -y -s [parameters('resolution')] -strict -2 {fileNameWithoutExtension}_[parameters('resolution')].mp4",
                    "resourceFiles": [
                        {
                            "blobSource": "{url}",
                            "filePath": "{fileName}"
                        }
                    ],
                    "outputFiles": [
                        {
                            "filePattern": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                            "destination": {
                                "autoStorage": {
                                    "path": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                                    "fileGroup": "ffmpeg-output"
                                }
                            },
                            "uploadOptions": {
                                "uploadCondition": "TaskSuccess"
                            }
                        }
                    ]
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
```

Als de naam van het sjabloonbestand _taak ffmpeg.json_, klikt u vervolgens de sjabloon als volgt starten:

```azurecli
az batch job create --template job-ffmpeg.json
```

## <a name="file-groups-and-file-transfer"></a>Bestandsgroepen en bestandsoverdracht

De meeste jobs en taken vereisen invoerbestanden en uitvoerbestanden wordt geproduceerd. Normaal gesproken worden invoer en uitvoerbestanden overgedragen, vanaf de client naar het knooppunt of vanuit het knooppunt aan de client. De extensie Azure Batch CLI isoleert opslag bestandsoverdracht en maakt gebruik van het opslagaccount dat is gemaakt door de standaardwaarde voor elk Batch-account.

Een bestandsgroep gelijkstaat aan een container waarin de Azure storage-account is gemaakt. De bestandsgroep misschien submappen.

De Batch CLI-uitbreiding bevat opdrachten voor het uploaden van bestanden van de client naar een opgegeven bestandsgroep en bestanden van de opgegeven bestandsgroep downloaden naar een client.

```azurecli
az batch file upload --local-path c:\source_videos\*.mp4 
    --file-group ffmpeg-input

az batch file download --file-group ffmpeg-output --local-path
    c:\output_lowres_videos
```

Groep van toepassingen en -taak sjablonen kunnen bestanden die zijn opgeslagen in bestandsgroepen moet worden opgegeven voor kopiëren naar de knooppunten van de pool- of uitschakelen knooppunten terug naar een groep van toepassingen. Bijvoorbeeld, in de taaksjabloon eerder hebt opgegeven, is de bestand groep 'ffmpeg-invoer' opgegeven voor de taak factory als de locatie van de video bronbestanden gekopieerd naar het knooppunt voor transcodering; de groep 'ffmpeg-uitvoer in een bestand' is de locatie op waarnaar de uitvoerbestanden getranscodeerd zijn gekopieerd vanaf het knooppunt waarop elke taak wordt uitgevoerd.

## <a name="summary"></a>Samenvatting

Alleen voor de Azure CLI momenteel ondersteuning voor het overbrengen sjabloon en een bestand zijn toegevoegd. Het doel is om uit te breiden de doelgroep die Batch om gebruikers die u hoeft niet gebruiken kunt te ontwikkelen code met de Batch-API's, zoals onderzoekers, IT-gebruikers, enzovoort. Zonder codering, kunnen gebruikers met kennis van Azure Batch en de toepassingen worden uitgevoerd door de Batch sjablonen voor het maken van toepassingen en -taak maken. Gebruikers zonder grondige kennis van de Batch- en de toepassingen kunnen de sjablonen gebruiken met de sjabloonparameters.

Uitproberen van de Batch-extensie voor de Azure CLI en geef ons feedback en suggesties, hetzij in de opmerkingen voor dit artikel of via de [Azure Batch-forum](https://social.msdn.microsoft.com/forums/azure/home?forum=azurebatch).

## <a name="next-steps"></a>Volgende stappen

- Zie het Batch-sjablonen blogbericht: [uitgevoerd op Azure Batch-taken met de Azure CLI – geen code vereist](https://azure.microsoft.com/en-us/blog/running-azure-batch-jobs-using-the-azure-cli-no-code-required/).
- Gedetailleerde documentatie voor installatie en het gebruik, voorbeelden en broncode zijn beschikbaar in de [Azure GitHub-opslagplaats](https://github.com/Azure/azure-batch-cli-extensions).
