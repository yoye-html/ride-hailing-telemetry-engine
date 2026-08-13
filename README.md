```mermaid
graph TD
    subgraph Producer Layer
        P[Telemetry Producer <br/> Python Emitter] -->|Key: driver_id| K
    end

    subgraph Streaming Backbone
        K[Apache Kafka Broker <br/> Topic: driver_telemetry <br/> 3 Partitions]
    end

    subgraph Consumer Group 1: Archival Pipeline
        K -->|Consumer Group: hdfs-writer| C1[In-Memory Dual-Trigger Buffer <br/> 1000 records OR 30 seconds]
        C1 -->|Batch Flush JSONL| HDFS[(Apache Hadoop HDFS <br/> /data/telemetry_raw)]
    end

    subgraph Consumer Group 2: Real-Time Analytics
        K -->|Consumer Group: realtime-congestion-detector| C2[Speed Drop Evaluation <br/> Threshold < 5 km/h]
        C2 -->|Terminal Output| Alerts[Live Congestion Alerts]
    end
# Ride-Hailing Geospatial Telemetry Engine (Addis Ababa)

## 📌 Overview
A high-throughput, distributed data ingestion pipeline simulating and capturing real-time GPS telemetry from a ride-hailing fleet in Addis Ababa. Designed to mitigate the HDFS "Small Files Problem," this architecture decouples real-time ingestion from historical storage using Apache Kafka and an in-memory dual-trigger batching consumer.

## 🏗️ System Architecture
* **Producer Layer:** Python-based telemetry emitter generating high-velocity GPS pings, keyed by `driver_id`.
* **Streaming Backbone:** Apache Kafka 3.9.2 (ZooKeeper Mode) ensuring strict intra-partition chronological ordering.
* **Consumer Group 1 (Archival):** Custom Python engine that buffers streaming events in memory, executing a dual-trigger flush (1,000 records OR 30 seconds) to Apache Hadoop HDFS.
* **Consumer Group 2 (Real-Time Analytics):** Independent Pub/Sub consumer monitoring the live stream for severe traffic congestion (speed < 5 km/h) without impacting the HDFS write path.

## 🛠️ Technology Stack
* **OS:** Ubuntu Linux (WSL2)
* **Languages:** Python 3.10 (kafka-python-ng)
* **Message Broker:** Apache Kafka / ZooKeeper
* **Distributed Storage:** Apache Hadoop (HDFS) 3.3.6
* **Data Format:** JSON / JSONL

## 💡 Engineering Defenses & Trade-offs
* **Partitioning Strategy:** Messages are keyed by `driver_id` to guarantee that spatial trajectory updates for individual vehicles are strictly ordered within the same partition.
* **Producer Acknowledgment:** Configured with `acks=1` to prioritize ingestion throughput and minimize latency for high-frequency GPS tracking.
* **HDFS Metadata Optimization:** Raw streams are buffered in memory to prevent NameNode RAM exhaustion (the Small Files Problem). A batch size of 1,000 reduces metadata footprint by 1000x compared to unbatched writes.

## 🚀 Execution Guide
*(List the commands to start Zookeeper, Kafka, HDFS, and your Python scripts here)*
