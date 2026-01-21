# Low-Level Design (LLD): Logic & Algorithms
**Reference:** PM-HYB-LLD-002
**Scope:** Decision Logic, Pricing & Rules

## 3. Provider Matching Logic
**Constraint:** Max 10km Radius. Prioritize Internal.

```mermaid
flowchart TD
    start([Start: Hybrid Request]) --> input[Input: Patient Lat/Long]
    input --> query[Query Provider Database]
    
    query --> filter1{Distance < 10km?}
    filter1 -- No --> discard[Exclude Provider]
    filter1 -- Yes --> list[Candidate List]
    
    list --> sort1{Sort Criteria}
    sort1 --> p1[Priority 1: Internal Staff]
    sort1 --> p2[Priority 2: Rated External > 4.5]
    sort1 --> p3[Priority 3: Nearest Neighbor]
    
    p1 --> result([Output: Sorted Provider List])
    p2 --> result
    p3 --> result
```

## 4. Pricing Decision Flow
**Constraint:** Internal = Fixed; External = Flexible.

```mermaid
flowchart TD
    A[Provider Selection] --> B{Provider Type?}
    
    B -->|Internal / Staff| C[Fetch Standard Rate Card]
    C --> C1[Zone A: ₹1000]
    C --> C2[Zone B: ₹700]
    C1 --> D[Final Price Displayed]
    C2 --> D
    
    B -->|External / Partner| E[Fetch Partner Configuration]
    E --> F{Set Own Price?}
    
    F -->|Yes| G[Display Provider Ask Price]
    F -->|No| H[Apply Platform Recommended Range]
    
    G --> I[Patient Accepts?]
    H --> I
    
    I -- Yes --> D
    I -- No --> J[Negotiation / Re-select]
```

## 5. Revenue Split Logic Flow
**Constraint:** Adaptive split based on Provider Type.
**Formula:** Total = Doctor_Fee + Provider_Fee + Platform_Commission

```mermaid
flowchart TD
    Transaction[Payment Captured] --> Check{Provider Type}
    
    Check -->|Internal| Split1[Standard Split]
    Split1 --> D1[Doctor: 20%]
    Split1 --> P1[Provider: Salary via Payroll]
    Split1 --> PL1[Platform: 80% (Gross Revenue)]
    
    Check -->|External| Split2[Marketplace Split]
    Split2 --> D2[Doctor: 15%]
    Split2 --> P2[Provider: 70% (Payout)]
    Split2 --> PL2[Platform: 15% (Commission)]
    
    D1 --> WalletDO[Doctor Wallet]
    PL1 --> WalletPL[Company Account]
    
    D2 --> WalletDO
    P2 --> WalletPR[Provider Wallet]
    PL2 --> WalletPL
```
