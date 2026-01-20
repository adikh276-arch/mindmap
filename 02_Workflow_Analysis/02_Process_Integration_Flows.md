# Workflow Analysis: Integration & Diversion Points
**Reference ID:** PM-HYB-FLOW-002  
**Date:** 2026-01-20  
**Scope:** Operational Workflow & Logic  

---

## 1. The Diversion Logic

The primary risk in hybrid models is "Process Forking," where the offline business runs separately from the online code. 

To maintain the **"One Doctor, One Plan"** principle, we identify a single diversion point: **Post-Prescription Delivery.**

*   **Before this point:** The flow is identical (Booking -> Video -> Plan).
*   **At this point:** A configuration switch determines execution mode.
*   **After this point:** Data converges back to the same Dashboard.

---

## 2. Current Workflow (Linear Online)

Currently, the system relies on a high-friction "Self-Start" mechanism.

**Critical Friction Point:** `Patient Execution`.
At this node, the system loses visibility. We do not know *how* the patient executes, or *if* they execute correctly, until they self-report.

```mermaid
flowchart TD
    %% Nodes
    A([Patient Entry]) -->|Mobile/Web| B{Booking Engine}
    B -->|Slot Allocation| C[**Doctor Consultation**<br/>Video Call via WebRTC]
    C -->|Diagnosis| D[**Care Plan Creation**<br/>Digital Exercise Prescription]
    
    D --> E{**Patient Execution**<br/>The 'Black Box'}
    E -->|Independent| F[Patient Opens App]
    F --> G[Patient Watches Video Content]
    G --> H[**Patient Attempts Exercise**<br/>High Cognitive Load / Uncertainty]
    H --> I[Patient Logs Completion]
    I --> J[Doctor Dashboard Review]

    %% Styling
    style E fill:#fff0f0,stroke:#d00,stroke-width:2px,stroke-dasharray: 5 5
    style H fill:#fff0f0,stroke:#d00
```

---

## 3. The Hybrid Fork (Reconfigured)

By introducing the conditional branch `IF mode == HYBRID`, we bridge the "Execution Gap" without changing the upstream booking or downstream diagnosis flows.

**The Assistant 'Inject' Strategy:**
The Assistant does not need a new app. They log in to the **Patient's View** (or a proxy view thereof) and execute the *same plan* that the patient would have seen.

```mermaid
flowchart TD
    %% Nodes
    A([Patient Entry]) --> B{Booking Engine}
    B -->|Unchanged| C[**Doctor Consultation**<br/>Video Call]
    C --> D[**Care Plan Creation**<br/>Same Object / Same Dashboard]
    
    D --> E{**Execution Mode?**<br/>Config Switch}
    
    E -->|Standard| F[Online-Only Flow]
    
    E -->|**Hybrid Config**| G[**Assistant Assigned**]
    G --> H[Assistant Visits Patient]
    H --> I[**Assisted Execution**<br/>Assistant Opens Existing App]
    
    I --> J[**Real-time Correction**<br/>Tactile Feedback & Safety]
    J --> K[**Verified Completion**<br/>Assistant Logs as Proxy]
    
    K --> L[Doctor Dashboard Review]
    F -.-> L

    %% Styling
    style E fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style G fill:#e8f5e9,stroke:#2e7d32
    style H fill:#e8f5e9,stroke:#2e7d32
    style I fill:#e8f5e9,stroke:#2e7d32
    style J fill:#e8f5e9,stroke:#2e7d32
    style K fill:#e8f5e9,stroke:#2e7d32
```

## 4. Technical Validation

*   **No New Booking Module:** The Assistant's schedule is slave to the Patient's slot.
*   **No New Clinical Data:** The Assistant inputs standard fields (Reps, Sets, Pain Level), just with higher veracity.
*   **Convergent Analytics:** The Doctor sees "Completed Plans" regardless of mode; the data schema remains uniform.
