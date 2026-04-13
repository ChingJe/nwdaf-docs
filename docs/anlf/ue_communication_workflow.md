# UE Communication Analytics Workflow

This document illustrates the high-level workflow of the NWDAF (AnLF) for UE Communication Analytics (`UE_COMM`), including subscription handling, data collection triggering, ML model provisioning, and periodic notification.

## Workflow Diagram

```mermaid
sequenceDiagram
    autonumber
    participant C as Consumer
    participant AnLF as NWDAF (AnLF)
    participant Res as GroupResolver
    participant SMF as SMF
    participant UPF as UPF (Data Source)
    participant MTLF as NWDAF (MTLF)
    participant IE as Inference Engine

    %% 1. Subscription Phase
    Note over C, AnLF: 1. Subscription Phase
    C->>AnLF: POST /events/subscriptions (UE Comm)
    activate AnLF
    AnLF->>AnLF: Validate Request & Defaults
    AnLF->>Res: Resolve Target (Group ID -> SUPIs)
    activate Res
    Res-->>AnLF: Return SUPI List
    deactivate Res

    %% 2. Setup Phase
    Note over AnLF, MTLF: 2. Setup Phase (Data Collection & Model)
    par Data Collection
        loop For each SUPI
            AnLF->>SMF: POST /subscriptions (Subscribe UPF Events)
            activate SMF
            SMF-->>AnLF: 201 Created (CorrelationId)
            deactivate SMF
        end
    and Model Provisioning
        AnLF->>MTLF: Request ML Model (Provisioning)
        activate MTLF
        MTLF-->>AnLF: Return Model Info & URL
        deactivate MTLF
        AnLF->>IE: Initialize Engine (Load Model)
        activate IE
        IE-->>AnLF: Engine Ready (Session ID)
        deactivate IE
    end
    AnLF-->>C: 201 Created (SubscriptionID)
    deactivate AnLF

    %% 3. Data Ingestion Phase
    Note over UPF, AnLF: 3. Data Ingestion Phase
    loop Continuous Traffic Data
        UPF->>AnLF: POST /upf-notify (Usage Report)
        activate AnLF
        AnLF->>AnLF: Normalize & Store in TrafficDataBucket
        deactivate AnLF
    end

    %% 4. Analytics & Notification Phase
    Note over C, IE: 4. Analytics & Notification Phase
    loop Every Notification Period
        AnLF->>AnLF: Scheduler Trigger
        activate AnLF
        AnLF->>AnLF: Fetch Aggregated Traffic Data
        AnLF->>IE: Predict(Traffic Features)
        activate IE
        IE-->>AnLF: Analytics Prediction (e.g., Abnormal Behavior)
        deactivate IE
        AnLF->>AnLF: Generate Analytics Report
        AnLF->>C: POST /notify (AnalyticsInfo)
        deactivate AnLF
    end
```

## Detailed Steps

1.  **Subscription**: Consumer sends a subscription request for `UE_COMMUNICATION` analytics. AnLF validates the request and uses `GroupResolver` to translate any Group IDs into a list of individual SUPIs.
2.  **Setup**:
    *   **Data Collection**: AnLF subscribes to the SMF for UPF events (Using `Supi` and `CorrelationId`) for each target UE.
    *   **Model Provisioning**: AnLF requests an ML model from the MTLF. Upon receiving the Model URL, it initializes the internal Inference Engine for future predictions.
3.  **Data Ingestion**: The UPF (simulated via SMF configuration or directly) sends periodic usage reports (`USER_DATA_USAGE_MEASURES`) to AnLF. AnLF stores this raw data, enriched with identifiers.
4.  **Notification**: When the reporting period defined in the consumer subscription expires, AnLF:
    *   Retrieves accumulated data.
    *   Calls the Inference Engine to `Predict`.
    *   Formats the result into an `AnalyticsInfo` notification.
    *   Sends the notification back to the Consumer.
