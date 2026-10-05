# Technical Design Specification (TDS)
**Project:** Grill & Go Digital Ordering System  

## 1. C4 Architecture Model

### C1: System Context Diagram
High-level view showing human actors, system boundaries, and external integrations.

```mermaid
flowchart TB
    Customer([Customer])
    KitchenStaff([Kitchen Staff])
    StoreManager([Store Manager])
    
    subgraph GrillAndGo [Grill & Go Ordering System]
        System((Grill & Go\nOrdering System))
    end
    
    PayNow((PayNow Gateway))
    BI((Power BI / Tableau))
    
    Customer -->|Scans QR, Orders & Pays| System
    KitchenStaff -->|Updates Order Status| System
    StoreManager -->|Toggles Stock, Views Reports| System
    System -->|Processes Payments| PayNow
    System -->|Reads Transactional Data| BI
```

### C2: Container Diagram
Technology stack choices and internal system containers.

```mermaid
flowchart TB
    Customer([Customer])
    KitchenStaff([Kitchen Staff])
    StoreManager([Store Manager])
    
    subgraph GrillAndGo [Grill & Go Ordering System]
        MobileFE["Mobile Web App (React.js / PWA)"]
        KDS["KDS Tablet App (React.js)"]
        API["API Backend (Node.js / Express)"]
        DB[("Relational DB (PostgreSQL)")]
    end
    
    PayNow((PayNow Gateway))
    BI((Power BI / Tableau))
    
    Customer -->|HTTPS| MobileFE
    KitchenStaff -->|HTTPS| KDS
    StoreManager -->|HTTPS| MobileFE
    MobileFE -->|REST/JSON| API
    KDS -->|REST/WebSocket| API
    API -->|SQL| DB
    API -->|REST/JSON| PayNow
    DB -->|Direct DB Connection| BI
```

---

## 2. UML Sequence Diagram (Order Placement & PayNow Flow)

```mermaid
sequenceDiagram
    actor C as Customer
    participant M as Mobile Browser
    participant A as API Backend
    participant P as PayNow Gateway
    participant D as Database
    participant K as KDS Tablet

    C->>M: 1. Selects items & customizations
    M->>A: 2. POST /api/v1/orders
    A->>A: 3. Calculate total & validate stock
    A->>P: 4. Request Payment (PayNow/CC)
    P-->>A: 5. Payment Success Response
    A->>D: 6. INSERT Order & Payment records
    D-->>A: 7. Confirm Save
    A->>K: 8. Push new order notification (WebSocket)
    A-->>M: 9. Return Order Confirmation
    M-->>C: 10. Display "Order Sent to Kitchen"
```

---

## 3. Database Schema (ERD)

```mermaid
erDiagram
    MENU_ITEMS {
        int id PK
        string name
        string description
        decimal price
        boolean is_available
    }
    ORDERS {
        int id PK
        int table_number
        decimal total_amount
        string status
        timestamp created_at
    }
    ORDER_ITEMS {
        int id PK
        int order_id FK
        int menu_item_id FK
        int quantity
        string customizations
        decimal price
    }
    PAYMENTS {
        int id PK
        int order_id FK
        decimal amount
        string payment_method
        string transaction_ref
        string status
        timestamp created_at
    }

    ORDERS ||--o{ ORDER_ITEMS : contains
    MENU_ITEMS ||--o{ ORDER_ITEMS : includes
    ORDERS ||--|| PAYMENTS : has
```


## 4. API Interface Specifications

### Endpoint 1: Place Order
**`POST /api/v1/orders`**
*Description: Creates a new order and processes immediate payment.*

**Request Payload:**
```json
{
  "table_number": 5,
  "items": [
    {
      "menu_item_id": 101,
      "quantity": 2,
      "customizations": "Medium rare, no onions"
    }
  ],
  "payment_method": "PAYNOW",
  "payment_token": "txn_123456789"
}
```

**Response Payload:**
```json
{
  "order_id": 9001,
  "status": "PAID_PENDING_KITCHEN",
  "total_amount": 45.50,
  "payment_status": "SUCCESS",
  "estimated_prep_time_mins": 15
}
```

### Endpoint 2: Update Inventory (Out of Stock Override)
**`PATCH /api/v1/menu/items/{item_id}`**
*Description: Allows Store Manager to toggle item availability in real-time.*

**Request Payload:**
```json
{
  "is_available": false
}
```

**Response Payload:**
```json
{
  "menu_item_id": 101,
  "name": "Ribeye Steak",
  "is_available": false,
  "updated_at": "2026-10-12T14:30:00Z"
}
```

---

## 5. Document Approval & Technical Sign-Off

By signing below, the undersigned engineering leads acknowledge that the architecture, schema, and API specifications detailed within this Technical Design Specification (TDS) are technically feasible, scalable to 30,000 monthly transactions, and approved for implementation.

| Approval Role | Engineer Name | Project Role |  Timestamp (SGT) | Digital Sign-Off (Git ID) |
| :--- | :--- | :--- | :--- | :--- | 
| **Lead Systems Analyst** | Kaveesha Lakruwan |  **APPROVED** | 2026-10-05 16:00 | @lakruwanb2004-lgtm |
| **Lead Software Engineer** |  Lesandu Dulneth | **APPROVED** | 2026-10-05 16:15 |  @lesandudulneth-cloud  |
