---
title: Verbinding maken met Azure Cosmos DB als een gegevensbron in Azure Machine Learning-Workbench | Microsoft Docs
description: Dit document bevat een voorbeeld over verbinding maken met Azure Cosmos DB via Azure Machine Learning Workbench
services: machine-learning
author: cforbe
ms.author: cforbe
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.workload: data-services
ms.custom: 
ms.devlang: 
ms.topic: article
ms.date: 09/11/2017
ms.openlocfilehash: d36b394a528dc4bc1b6e0a9e0e5dbde728cbee1b
ms.sourcegitcommit: 0b02e180f02ca3acbfb2f91ca3e36989df0f2d9c
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/05/2018
---
# <a name="connecting-to-azure-cosmos-db-as-a-data-source"></a>Verbinding maken met Azure Cosmos DB als een gegevensbron
Dit artikel bevat een python voorbeeld kunt u verbinding maken met de Cosmos-database in de Azure Machine Learning-Workbench.

## <a name="load-azure-cosmos-db-data-into-data-preparation"></a>Azure DB die Cosmos-gegevens laden in het voorbereiden van gegevens

Een nieuwe stroom van gegevens op basis van een script maken en het volgende script vervolgens gebruiken voor het laden van de gegevens uit Azure Cosmos DB. 

```python
import pydocumentdb
import pydocumentdb.document_client as document_client

import pandas as pd

config = { 
    'ENDPOINT': '<Endpoint>',
    'MASTERKEY': '<Key>',
    'DOCUMENTDB_DATABASE': '<DBName>',
    'DOCUMENTDB_COLLECTION': '<collectionname>'
};

# Initialize the Python DocumentDB client.
client = document_client.DocumentClient(config['ENDPOINT'], {'masterKey': config['MASTERKEY']})

# Read databases and take first since id should not be duplicated.
db = next((data for data in client.ReadDatabases() if data['id'] == config['DOCUMENTDB_DATABASE']))

# Read collections and take first since id should not be duplicated.
coll = next((coll for coll in client.ReadCollections(db['_self']) if coll['id'] == config['DOCUMENTDB_COLLECTION']))

docs = client.ReadDocuments(coll['_self'])

df = pd.DataFrame(list(docs))
```

## <a name="other-data-source-connections"></a>Andere gegevensbronverbindingen
Lees voor andere voorbeelden [voorbeeld extra bron gegevensverbindingen](data-prep-appendix8-sample-source-connections-python.md)
