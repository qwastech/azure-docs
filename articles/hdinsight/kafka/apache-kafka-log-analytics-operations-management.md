---
title: Azure Monitor logs for Apache Kafka - Azure HDInsight 
description: Learn how to use Azure Monitor logs to analyze logs from Apache Kafka cluster on Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/04/2019
---

# Analyze logs for Apache Kafka on HDInsight

Learn how to use Azure Monitor logs to analyze logs generated by Apache Kafka on HDInsight.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## Enable Azure Monitor logs for Apache Kafka

The steps to enable Azure Monitor logs for HDInsight are the same for all HDInsight clusters. Use the following links to understand how to create and configure the required services:

1. Create a Log Analytics workspace. For more information, see the [Logs in Azure Monitor](../../azure-monitor/platform/data-platform-logs.md) document.

2. Create a Kafka on HDInsight cluster. For more information, see the [Start with Apache Kafka on HDInsight](apache-kafka-get-started.md) document.

3. Configure the Kafka cluster to use Azure Monitor logs. For more information, see the [Use Azure Monitor logs to monitor HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) document.

> [!IMPORTANT]  
> It may take around 20 minutes before data is available for Azure Monitor logs.

## Query logs

1. From the [Azure portal](https://portal.azure.com), select your Log Analytics workspace.

2. From the left menu, under **General**, select **Logs**. From here, you can search the data collected from Kafka. Enter a query in the query window and then select **Run**. The following are some example searches:

* Disk usage:

    ```kusto
    Perf
    | where ObjectName == "Logical Disk" and CounterName == "Free Megabytes" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* CPU usage:

    ```kusto
    Perf 
    | where CounterName == "% Processor Time" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Incoming messages per second: (Replace `your_kafka_cluster_name` with your cluster name.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-MessagesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s, bin(TimeGenerated, 1h)
    ```

* Incoming bytes per second: (Replace `wn0-kafka` with a worker node host name.)

    ```kusto
    metrics_kafka_CL 
    | where HostName_s == "wn0-kafka" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

* Outgoing bytes per second: (Replace `your_kafka_cluster_name` with your cluster name.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesOutPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

    You can also enter `*` to search all types logged. Currently the following logs are available for queries:

    | Log type | Description |
    | ---- | ---- |
    | log\_kafkaserver\_CL | Kafka broker server.log |
    | log\_kafkacontroller\_CL | Kafka broker controller.log |
    | metrics\_kafka\_CL | Kafka JMX metrics |

    ![Apache kafka log analytics cpu usage](./media/apache-kafka-log-analytics-operations-management/apache-kafka-cpu-usage.png)

## Next steps

For more information on Azure Monitor, see [Azure Monitor overview](../../log-analytics/log-analytics-get-started.md), and [Query Azure Monitor logs to monitor HDInsight clusters](../hdinsight-hadoop-oms-log-analytics-use-queries.md).

For more information on working with Apache Kafka, see the following documents:

* [Mirror Apache Kafka between HDInsight clusters](apache-kafka-mirroring.md)
* [Increase the scale of Apache Kafka on HDInsight](apache-kafka-scalability.md)
* [Use Apache Spark streaming (DStreams) with Apache Kafka](../hdinsight-apache-spark-with-kafka.md)
* [Use Apache Spark structured streaming with Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)
