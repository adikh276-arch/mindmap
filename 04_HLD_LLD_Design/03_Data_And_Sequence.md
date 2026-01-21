# Low-Level Design (LLD): Data & Sequences
**Reference:** PM-HYB-LLD-003
**Scope:** Database Schema & Transaction Flow

## 6. Payment Sequence Diagram
**Flow:** Booking -> Escrow -> Service Delivery -> Payout

```mermaid
sequenceDiagram
    participant P as Patient
    participant SYS as Platform/Server
    participant PG as Payment Gateway
    participant PROV as Provider (Ext)
    participant DOC as Doctor

    P->>SYS: Selects Hybrid Session (Costs ₹800)
    SYS->>PG: Initialize Transaction (Capture ₹800)
    PG-->>P: Request Payment
    P->>PG: Completes Payment
    PG-->>SYS: Success Webhook
    
    SYS->>SYS: Create Transaction Record (Status: HELD)
    SYS->>PROV: Notify "New Booking Confirmed"
    
    Note over P, PROV: Service Delivery Day
    
    PROV->>SYS: Mark Session Complete (Upload Verification)
    SYS->>DOC: Request Clinical Sign-off
    DOC->>SYS: Approve Session log
    
    SYS->>SYS: Trigger Payout Logic
    SYS->>PG: Transfer ₹560 to Provider (70%)
    SYS->>PG: Transfer ₹120 to Doctor (15%)
    SYS->>SYS: Retain ₹120 (15% Platform Fee)
    
    SYS-->>P: Send Receipt
```

## 7. Data Model Diagram (Entity Relationships)
**Key Change:** `Appointment` entity gains `mode` and `provider_id`. No new tables needed for core logic, just relationship linking.

```mermaid
erDiagram
    Users ||--o{ Roles : has
    Users {
        string user_id PK
        string name
        string email
        float wallet_balance
    }
    
    Roles {
        string role_id PK
        string user_id FK
        enum type "DOCTOR, PATIENT, PROVIDER_INT, PROVIDER_EXT"
        json config "Rates, Availability"
    }

    ProviderServiceArea {
        string id PK
        string role_id FK
        geography location
        float radius_km
    }

    CarePlan ||--o{ Appointment : contains
    CarePlan {
        string plan_id PK
        string doctor_id FK
        string patient_id FK
        string diagnosis
    }
    
    Appointment {
        string appt_id PK
        string plan_id FK
        datetime scheduled_time
        enum mode "ONLINE, HYBRID"
        string assigned_provider_id FK "Nullable"
        enum status "SCHEDULED, COMPLETED, VERIFIED"
    }
    
    Transaction {
        string txn_id PK
        string appt_id FK
        float amount
        json split_details "{doc: 15, prov: 70}"
        enum status "HELD, SETTLED"
    }

    Roles ||--o{ ProviderServiceArea : defines
    Roles ||--o{ Appointment : fulfills
    Appointment ||--o{ Transaction : generates
```
