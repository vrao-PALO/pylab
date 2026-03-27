# NAPTA Architecture - Corrected Data Flow
## _Accurate Single PostgreSQL Server Per Environment with Logical Separation_

---

## **High-Level Architecture (Mermaid Diagram)**

```mermaid
graph TB
    subgraph "Identity & Auth"
        ENTRA["🔐 Entra ID<br/>SSO + RBAC + MFA"]
    end

    subgraph "CI/CD Pipeline"
        GH["📦 GitHub Actions"]
        ARM["Azure Resource Manager<br/>(Infrastructure Deploy - OIDC)"]
        FUNCEP["Protected Function Endpoints<br/>(Schema/Data Ops - Entra Bearer Tokens)"]
    end

    subgraph "External Data Sources"
        direction LR
        EXT1["🔄 External SaaS API"]
        EXT2["📊 SharePoint Graph API"]
        EXT3["📋 MOM Data Source"]
    end

    subgraph "Development Environment"
        direction TB
        DEVFUNC["⚙️ ETL Function App<br/>+ Migration Function App"]
        DEVNAT["🚪 NAT Gateway<br/>(Outbound only)"]
        DEVDB["💾 PostgreSQL Server<br/>ONE Instance Per Environment<br/><br/>├─ Logical Separation Via:<br/>├─ Tables (by classification)<br/>├─ Views (for masking)<br/>├─ PostgreSQL Roles (RBAC)<br/>├─ Row-Level Security (RLS)"]
        DEVKV["🔑 Key Vault"]
        DEVST["📦 Storage"]
    end

    subgraph "Production Environment"
        direction TB
        PRODFUNC["⚙️ ETL Function App<br/>+ Migration Function App"]
        PRODNAT["🚪 NAT Gateway<br/>(Outbound only)"]
        PRODDB["💾 PostgreSQL Server<br/>ONE Instance Per Environment<br/><br/>├─ Data Classification:<br/>├─ 🟢 INTERNAL (tables)<br/>├─ 🟡 CONFIDENTIAL (tables)<br/>├─ 🔴 FINANCIAL (tables)<br/>├─ ⚫ AUDIT (tables)<br/>├─ Role-Based Access<br/>├─ Audit Logging<br/>├─ Field-Level Masking"]
        PRODKV["🔑 Key Vault"]
        PRODST["📦 Storage"]
    end

    subgraph "Observability Stack"
        direction LR
        AI["📊 Application Insights"]
        LA["📝 Log Analytics"]
        MON["📡 Azure Monitor"]
        ACT["🔔 Action Groups"]
    end

    subgraph "SIEM & Alerts"
        direction LR
        SIEM["🛡️ Microsoft Sentinel<br/>(or equivalent SIEM)<br/>Positioned DOWNSTREAM"]
        ALERTS["⚠️ Security Alerts<br/>Email / Teams / PagerDuty"]
    end

    subgraph "Data Consumers (Read-Side Access Only)"
        direction TB
        BI["📈 BI Tools<br/>(PowerBI, Tableau)<br/>Role: role_bi_read<br/>Access: INTERNAL tables"]
        HR["👥 HR Users<br/>(via SSO)<br/>Role: role_hr_read<br/>Access: CONFIDENTIAL tables"]
        FIN["💰 Finance Users<br/>(via SSO)<br/>Role: role_finance_read<br/>Access: FINANCIAL tables"]
        AUD["✓ Audit Team<br/>(via SSO)<br/>Role: role_audit_read<br/>Access: AUDIT tables"]
    end

    %% Connections - Authentication
    ENTRA -->|SSO Token| HR
    ENTRA -->|SSO Token| FIN
    ENTRA -->|SSO Token| AUD
    ENTRA -->|Managed Identity| DEVFUNC
    ENTRA -->|Managed Identity| PRODFUNC

    %% Connections - CI/CD
    GH -->|OIDC| ARM
    GH -->|Entra Token| FUNCEP
    ARM -->|Deploy| DEVFUNC
    ARM -->|Deploy| PRODFUNC

    %% Connections - Data Ingestion
    DEVFUNC -->|Ingest| EXT1
    DEVFUNC -->|Ingest| EXT2
    DEVFUNC -->|Ingest| EXT3
    PRODFUNC -->|Ingest| EXT1
    PRODFUNC -->|Ingest| EXT2
    PRODFUNC -->|Ingest| EXT3

    %% Connections - Outbound via NAT
    DEVFUNC --> DEVNAT
    DEVNAT -->|Controlled Exit| EXT1
    DEVNAT -->|Controlled Exit| EXT2
    DEVNAT -->|Controlled Exit| EXT3

    PRODFUNC --> PRODNAT
    PRODNAT -->|Controlled Exit| EXT1
    PRODNAT -->|Controlled Exit| EXT2
    PRODNAT -->|Controlled Exit| EXT3

    %% Connections - Secrets & Storage
    DEVFUNC -.->|Fetch Secrets| DEVKV
    DEVFUNC -.->|Read/Write| DEVST
    PRODFUNC -.->|Fetch Secrets| PRODKV
    PRODFUNC -.->|Read/Write| PRODST

    %% Connections - Database Access (ROLE-BASED)
    DEVFUNC -->|Write (Insert/Update)| DEVDB
    PRODFUNC -->|Write (Insert/Update)| PRODDB

    HR -->|Read (role_hr_read)| PRODDB
    FIN -->|Read (role_finance_read)| PRODDB
    AUD -->|Read (role_audit_read)| PRODDB
    BI -->|Read (role_bi_read)| PRODDB

    %% Connections - Monitoring
    DEVFUNC -.->|Logs| AI
    PRODFUNC -.->|Logs| AI
    AI -->|Ingest| LA
    LA -->|Monitor| MON
    MON -->|Actions| ACT

    %% Connections - SIEM (Downstream)
    LA -->|Forward Logs| SIEM
    MON -->|Forward Logs| SIEM
    SIEM -->|Trigger| ALERTS

    %% Styling
    classDef secure fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    classDef warning fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    classDef critical fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    classDef info fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff

    class ENTRA,DEVDB,PRODDB secure
    class DEVFUNC,PRODFUNC,AI,LA,MON info
    class GH,ARM,FUNCEP info
    class SIEM warning
```

---

## **Data Flow: From Ingest to Access**

### **1️⃣ INGEST PHASE (External → ETL Function App)**

```
External System                ETL Function App           NAT Gateway
     ⬇️                             ⬇️                          ⬇️
SharePoint API    ────HTTPS──→  ETL App                  Controlled
    MOM Data      ────HTTPS──→  (Managed Identity)       Outbound
  SaaS APIs       ────HTTPS──→  (Validates & Transforms)  Only
                                        ⬇️
                            Parse, validate, enrich
                                        ⬇️
                            Determine classification
                        (INTERNAL/CONFIDENTIAL/FINANCIAL)
```

---

### **2️⃣ PROCESS PHASE (ETL → PostgreSQL)**

```
ETL Function App
        ⬇️
    WRITE to PostgreSQL
        ⬇️
┌─────────────────────────────────────────────────────┐
│            ONE PostgreSQL Server (per env)         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  🟢 INTERNAL DATA (Public Data)                    │
│  ├─ employees (name, department, title)             │
│  ├─ organizations (company info, contact)           │
│  └─ audit_logs (access trail - anonymized)         │
│                                                     │
│  🟡 CONFIDENTIAL DATA (Restricted)                 │
│  ├─ employee_details (SSN, address, phone)         │
│  ├─ performance_reviews (ratings, feedback)        │
│  └─ health_info (if HIPAA-applicable)              │
│                                                     │
│  🔴 FINANCIAL DATA (Highly Restricted)             │
│  ├─ payroll (salary, benefits, deductions)         │
│  ├─ invoices (vendor, amounts, contracts)          │
│  └─ budgets (allocations, forecasts)               │
│                                                     │
│  ⚫ AUDIT DATA (Compliance-Critical)                │
│  ├─ data_access_log (WHO accessed WHAT, WHEN)     │
│  ├─ failed_access_attempts                         │
│  ├─ schema_changes                                 │
│  └─ compliance_events                              │
│                                                     │
└─────────────────────────────────────────────────────┘

LOGICAL SEPARATION MECHANISMS:
├─ Tables: Each data class in separate tables
├─ Views: Masked views hide PII (e.g., salary → range)
├─ Roles: HR user gets role_hr_read (CONFIDENTIAL only)
├─ RLS Policies: Row-level rules (user sees only their dept)
└─ Audit Trigger: Every SELECT/INSERT/UPDATE logged
```

---

### **3️⃣ ACCESS PHASE (PostgreSQL → Consumers via RBAC)**

```
┌─── PRODUCTION PostgreSQL ───┐
│ (Encrypted, Backed-up)      │
│ (24/7 Monitored)            │
└────────────────────────────┘
          ⬇️
┌─────────────────────────────────────────────────────┐
│         PostgreSQL Role-Based Access Control       │
├─────────────────────────────────────────────────────┤
│                                                     │
│ role_hr_read     ──→ SELECT on CONFIDENTIAL only   │
│  ⬇️ Assigned to:      └─ Cannot see FINANCIAL      │
│  ├─ HR Manager                                     │
│  ├─ HR Analyst         Query: "SELECT * FROM       │
│  └─ HR Business Partner    employee_details"      │
│                        Result: ✅ Allowed          │
│                                                     │
│ role_finance_read ──→ SELECT on FINANCIAL only    │
│  ⬇️ Assigned to:      └─ Cannot see CONFIDENTIAL   │
│  ├─ Finance Manager                                │
│  ├─ Finance Analyst    Query: "SELECT * FROM       │
│  └─ Accounting Clerk       payroll"                │
│                        Result: ✅ Allowed          │
│                                                     │
│ role_audit_read  ──→ SELECT on AUDIT only         │
│  ⬇️ Assigned to:      └─ Can see access trails     │
│  ├─ Audit Manager      Query: "SELECT * FROM       │
│  └─ Compliance Officer    data_access_log"        │
│                        Result: ✅ Allowed          │
│                                                     │
│ role_bi_read    ──→ SELECT on INTERNAL only       │
│  ⬇️ Assigned to:      └─ Cannot see sensitive data │
│  └─ BI Analyst         Query: "SELECT * FROM       │
│                            organizations"          │
│     Via PowerBI            Result: ✅ Allowed      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### **4️⃣ AUDIT PHASE (Database → Log Analytics → SIEM)**

```
User Query          PostgreSQL Audit Log        Alert
     ⬇️                   ⬇️                      ⬇️

SELECT * FROM    ──→  Logged:                Detected:
  payroll        ├─ user_id: alice@acme.com  ├─ Alice (Finance)
                 ├─ timestamp: 2026-03-26    ├─ Queried payroll
  WHERE year=    ├─ action: SELECT           ├─ At 02:15 AM
  2025 AND       ├─ table: payroll           └─ (OUTSIDE HOURS!)
  salary > 200k  ├─ row_count: 500           
                 ├─ query_hash: abc123       ⚠️ ALERT TRIGGERED:
                 └─ result: SUCCESS          "After-hours access"
                        ⬇️                   ⬇️
                 Log Analytics               Microsoft Sentinel
                 (Centralized)               (SIEM)
                        ⬇️
                  ⏰ Security Team Gets
                     Email/Teams Notification
```

---

## **Key Design Principles**

### **1. Single PostgreSQL Server Per Environment**
- ✅ NOT separate instances per data class
- ✅ Cost-efficient, simpler to manage
- ✅ Logical separation via roles + RLS + views
- ✅ Lower operational overhead

### **2. Managed Identity (Per Function App)**
- ✅ ETL Function App → Separate identity
- ✅ Migration Function App → Separate identity
- ✅ Each scoped to: Key Vault, Storage, Graph API
- ✅ NO hardcoded credentials

### **3. Role-Based Access Control (Data Layer)**
- ✅ HR users → `role_hr_read` → CONFIDENTIAL tables only
- ✅ Finance users → `role_finance_read` → FINANCIAL tables only
- ✅ Audit team → `role_audit_read` → AUDIT tables only
- ✅ BI analysts → `role_bi_read` → INTERNAL (masked) tables only

### **4. Two GitHub Actions Paths**
- 🔵 **Infrastructure Deploy:** GitHub → OIDC → Azure Resource Manager (ARM)
- 🟠 **Schema/Data Ops:** GitHub → Entra Bearer Token → Protected Function Endpoints

### **5. Observability (Full Stack)**
- ✅ Application Insights (telemetry)
- ✅ Log Analytics (centralization)
- ✅ Azure Monitor (alerting)
- ✅ Microsoft Sentinel (SIEM, positioned downstream)
- ⏳ Teams/Email (notifications) — TODO for prod

### **6. Consumers Access via Read-Side Only**
- ✅ No direct connections to raw backend objects
- ✅ All access through role-scoped permissions
- ✅ BI accesses masked views (no PII)
- ✅ HR/Finance/Audit cannot modify data

---

## **Data Classification & Access Rules**

| Data Class | Sensitivity | Storage | Who Can Access | Example Tables | Masking |
|------------|-------------|---------|-----------------|-----------------|----------|
| 🟢 **INTERNAL** | Public | One PostgreSQL | BI Analysts, Execs | employees, organizations | No (public data) |
| 🟡 **CONFIDENTIAL** | High | One PostgreSQL | HR Team, Audit | employee_details, perf_reviews | SSN (XXX-XX-####), Address (city only) |
| 🔴 **FINANCIAL** | Highly Restricted | One PostgreSQL | Finance Team, Audit | payroll, invoices, budgets | Salary (ranges), Bank details (hidden) |
| ⚫ **AUDIT** | Critical | One PostgreSQL | Audit, CISO | data_access_log, failed_attempts | None (must be transparent) |

---

## **Security Controls in Place**

| Layer | Control | Implementation |
|-------|---------|-----------------|
| **Network** | Private VNet + Private Endpoints | Azure VNet, private links to PG/KV/Storage |
| **Network** | NAT Gateway (Outbound only) | All external calls via single NAT |
| **Authentication** | Managed Identity | Per Function App, no secrets hardcoded |
| **Authentication** | Entra ID SSO | Users log in once; tokens used for DB access |
| **Authorization** | Infrastructure RBAC | Azure roles (Contributor, Owner) |
| **Authorization** | Data-Layer RBAC | PostgreSQL roles (role_hr_read, etc.) |
| **Authorization** | Row-Level Security | Users see only authorized rows |
| **Data Protection** | Encryption at Rest | Azure-managed keys (AES-256) |
| **Data Protection** | Encryption in Transit | TLS 1.2+ for all connections |
| **Data Protection** | Column Masking | Views hide sensitive fields |
| **Audit** | Audit Logging | PostgreSQL triggers on all data access |
| **Audit** | SIEM Integration | Logs forwarded to Sentinel for analysis |
| **Monitoring** | 24/7 Alerts | Anomaly detection via Sentinel detection rules |

---

## **Sample Access Scenario**

### **Scenario: Alice (HR Manager) Needs to See Employee Salary**

```
┌─ ALICE logs in with Entra ID (SSO)
│
├─ Entra provides token (includes role: "HR_MANAGER")
│
├─ Alice connects to PostgreSQL using token
│
├─ PostgreSQL checks: 
│    └─ Token → maps to role_hr_read
│    └─ role_hr_read → has SELECT on CONFIDENTIAL tables
│
├─ Alice queries: SELECT employee_id, name, salary FROM payroll
│
├─ PostgreSQL evaluates Row-Level Security:
│    └─ RLS Policy: "Sales dept employees can view only Sales dept salary"
│    └─ Alice is in "HR", so she sees ALL employees
│
├─ Result rows returned:
│    ├─ emp_001, John Smith, $95,000
│    ├─ emp_002, Jane Doe, $102,000
│    └─ ... (10,000 rows total)
│
├─ Audit log triggered:
│    ├─ timestamp: 2026-03-26 14:32:15
│    ├─ user_id: alice@acme.com
│    ├─ action: SELECT
│    ├─ table: payroll
│    ├─ row_count: 10000
│    ├─ query_hash: xyz789
│    └─ result: SUCCESS
│
└─ ⚠️ ALERTS (if triggered):
   ├─ IF row_count > 5000 in BI portal: "Large data export detected"
   ├─ IF Alice usually doesn't access payroll: "Unusual access pattern"
   └─ IF query ran at 02:15 AM: "After-hours data access"
```

---

## **Comparison: Current vs. Target Architecture**

| Aspect | Current (Design) | Target (Post-Implementation) |
|--------|------------------|------------------------------|
| **Data Storage** | ✅ Single PostgreSQL (correct) | ✅ Maintained |
| **Access Control** | ❌ NO roles; all users same access | ✅ Role-based (hr_read, fin_read, etc.) |
| **Audit Logging** | ❌ Table exists, inactive | ✅ Real-time, SIEM forwarding |
| **PII Protection** | ❌ Visible to all users | ✅ Masked views + DDM |
| **SIEM Integration** | ❌ Not started | ✅ Microsoft Sentinel active |
| **CI/CD Security** | ⚠️ OIDC only (partial) | ✅ Branch protection + code scanning |
| **Observability** | ⚠️ Logs collected, not analyzed | ✅ Automated detection + alerts |

---

## **Implementation Checklist**

Phase 1 (45 Days — Critical Fixes):
- [ ] Enable PostgreSQL audit logging
- [ ] Branch protection on main/prod
- [ ] Code scanning (SAST) in CI/CD
- [ ] SIEM setup (Sentinel)
- [ ] Security sign-off

Phase 2 (90 Days — RBAC & User Access):
- [ ] Create PostgreSQL roles
- [ ] Assign users to roles
- [ ] Implement RLS policies
- [ ] Test RBAC (cross-team verification)
- [ ] Production deployment

Phase 3 (180 Days — Data Security):
- [ ] Field-level masking (PII)
- [ ] Data retention policies
- [ ] DPA finalized
- [ ] Disaster recovery tested
- [ ] Penetration test

---

**Version:** 1.0  
**Last Updated:** March 26, 2026  
**Next Review:** Q2 2026 (Post-Implementation)
