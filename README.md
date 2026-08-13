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
