---
title: Meld u Analytics voor Apache Kafka - Azure HDInsight | Microsoft Docs
description: Informatie over het gebruik van logboekanalyse logboeken van Apache Kafka-cluster in Azure HDInsight analyseren.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/30/2018
ms.author: larryfr
ms.openlocfilehash: a373ef5cc71d5ae69c83555dc71525aa2188233e
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 04/16/2018
---
# <a name="analyze-logs-for-apache-kafka-on-hdinsight"></a>Logboeken voor Apache Kafka op HDInsight analyseren

Informatie over het gebruik van logboekanalyse logboeken die worden gegenereerd door Apache Kafka op HDInsight analyseren.

## <a name="enable-log-analytics-for-kafka"></a>Log Analytics voor Kafka inschakelen

De stappen voor het inschakelen van logboekanalyse voor HDInsight zijn hetzelfde voor alle HDInsight-clusters. Gebruik de volgende koppelingen om te begrijpen hoe maken en configureren van de vereiste services:

1. Maak een werkruimte voor logboekanalyse. Zie voor meer informatie de [aan de slag met een werkruimte voor logboekanalyse](https://docs.microsoft.com/azure/log-analytics) document.

2. Maak een Kafka op HDInsight-cluster. Zie voor meer informatie de [beginnen met Apache Kafka op HDInsight](apache-kafka-get-started.md) document.

3. Configureer het cluster Kafka Log Analytics gebruiken. Zie voor meer informatie de [gebruik logboekanalyse voor het bewaken van HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) document.

    > [!NOTE]
    > U kunt ook configureren met het cluster Log Analytics gebruiken met behulp van de `Enable-AzureRmHDInsightOperationsManagementSuite` cmdlet. Deze cmdlet vereist de volgende informatie:
    >
    > * De naam van het HDInsight-cluster.
    > * De ID van de werkruimte voor logboekanalyse. U kunt de werkruimte-ID vinden in de werkruimte voor logboekanalyse.
    > * De primaire sleutel voor de verbinding met logboekanalyse. Ga voor de primaire sleutel in uw Log Analytics-exemplaar selecteren en vervolgens __OMS-Portal__. Selecteer in de OMS-Portal __instellingen__, __verbonden bronnen__, en vervolgens __Linux-Servers__.


> [!IMPORTANT]
> Duurt ongeveer twintig minuten voordat de gegevens beschikbaar voor logboekanalyse.

## <a name="query-logs"></a>Querylogboeken

1. Van de [Azure-portal](https://portal.azure.com), selecteer uw werkruimte voor logboekanalyse.

2. Selecteer __Meld zoeken__. Hier kunt kunt u de gegevens die worden verzameld van Kafka zoeken. Hier volgen enkele voorbeelden van zoekopdrachten:

    * Schijfgebruik: `Type=Perf ObjectName="Logical Disk" (CounterName="Free Megabytes")  InstanceName="_Total" Computer='hn*-*' or Computer='wn*-*' | measure avg(CounterValue) by   Computer interval 1HOUR`
    * CPU-gebruik: `Type:Perf CounterName="% Processor Time" InstanceName="_Total" Computer='hn*-*' or Computer='wn*-*' | measure avg(CounterValue) by Computer interval 1HOUR`
    * Binnenkomende berichten per seconde: `Type=metrics_kafka_CL ClusterName_s="your_kafka_cluster_name" InstanceName_s="kafka-BrokerTopicMetrics-MessagesInPerSec-Count" | measure avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s interval 1HOUR`
    * Binnenkomende bytes per seconde: `Type=metrics_kafka_CL HostName_s="wn0-kafka" InstanceName_s="kafka-BrokerTopicMetrics-BytesInPerSec-Count" | measure avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) interval 1HOUR`
    * Uitgaande bytes per seconde: `Type=metrics_kafka_CL ClusterName_s="your_kafka_cluster_name" InstanceName_s="kafka-BrokerTopicMetrics-BytesOutPerSec-Count" |  measure avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) interval 1HOUR`

    > [!IMPORTANT]
    > De querywaarden vervangt door uw specifieke gegevens van de cluster. Bijvoorbeeld: `ClusterName_s` moet worden ingesteld op de naam van het cluster. `HostName_s` moet worden ingesteld op de domeinnaam van een knooppunt van de werknemer in het cluster.

    U kunt ook opgeven `*` om te zoeken van alle typen in het logboek geregistreerd. De volgende logboeken zijn momenteel beschikbaar voor query's:

    | Logboektype | Beschrijving |
    | ---- | ---- |
    | log\_kafkaserver\_CL | Kafka broker server.log |
    | log\_kafkacontroller\_CL | Kafka broker controller.log |
    | metrische gegevens\_kafka\_CL | JMX Kafka metrische gegevens |

    ![Afbeelding van de CPU-gebruik zoekopdracht](./media/apache-kafka-log-analytics-operations-management/kafka-cpu-usage.png)
 
 ## <a name="next-steps"></a>Volgende stappen

Zie voor meer informatie over logboekanalyse de [aan de slag met een werkruimte voor logboekanalyse](../../log-analytics/log-analytics-get-started.md) document.

Zie de volgende documenten voor meer informatie over het werken met Kafka:

 * [Mirror Kafka tussen HDInsight-clusters](apache-kafka-mirroring.md)
 * [Vergroot de schaalbaarheid van Kafka in HDInsight](apache-kafka-scalability.md)
 * [Spark-streaming (DStreams) met Kafka gebruiken](../hdinsight-apache-spark-with-kafka.md)
 * [Spark gestructureerd streamen met Kafka gebruiken](../hdinsight-apache-kafka-spark-structured-streaming.md)
