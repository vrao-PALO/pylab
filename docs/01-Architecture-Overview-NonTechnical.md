# NAPTA Integration Architecture Overview
## _For Non-Technical Stakeholders (HR, Finance, Leadership)_

---

## **What is NAPTA?**

NAPTA is our **secure data integration solution** that brings employee and financial data from multiple sources (SharePoint, external systems, MOM) into a centralized system where HR, Finance, and Audit teams can access it safely for reporting and analytics.

---

## **In Simple Terms: How Does It Work?**

### **Step 1: Data Collection (Inbound)**
- External systems (SharePoint, HR tools, Finance systems) send data to us
- Think of it like multiple mailboxes sending letters to our office
- All communication is secured and encrypted

### **Step 2: Data Processing (The "Smart Sort")**
- Our system automatically receives and organizes the data
- Different types of data (payroll, employee records, audit logs) are separated based on sensitivity
- This is done in secure, isolated environments for **Development** and **Production**

### **Step 3: Data Storage (The Vault)**
- Data is stored in a **single secure database per environment** (like one filing cabinet)
- Inside this cabinet, files are organized by **sensitivity levels**:
  - 🟢 **INTERNAL** — General company info (accessible to BI Tools)
  - 🟡 **CONFIDENTIAL** — Sensitive employee/department data (accessible only to HR & Audit)
  - 🔴 **FINANCIAL** — Payroll, budgets, contracts (accessible only to Finance)
  - ⚫ **AUDIT** — Compliance logs and access trails (accessible only to Audit)

### **Step 4: Access Control (The Security Guard)**
- Each user group (HR, Finance, Audit, BI team) has a **unique access key** (Entra ID SSO)
- Users can only see data relevant to their role—nothing more, nothing less
- All access is **logged and monitored** for compliance

### **Step 5: Reporting & Analytics**
- HR, Finance, Audit, and BI teams access the data through their own secure portals
- They can generate reports without seeing other departments' sensitive data
- Example: Finance can see payroll data, but HR cannot

### **Step 6: Monitoring & Alerts**
- Our security team monitors all activity 24/7
- If something suspicious happens, alerts are sent immediately
- All actions are recorded for audit purposes

---

## **Key Features You Should Know**

| Feature | What It Does | Why It Matters |
|---------|-------------|-----------------|
| **Encrypted Storage** | All data is locked with military-grade encryption | Protects against theft and breaches |
| **Role-Based Access** | Users only see data for their job | HR doesn't see Finance data; Finance doesn't see HR data |
| **Audit Logging** | Every access is recorded | Proves compliance with regulations (GDPR, SOX, etc.) |
| **Separate Environments** | DEV for testing, PROD for real use | Changes are tested before going live; stability guaranteed |
| **24/7 Monitoring** | Security team watches for threats | Early detection = quick response |
| **Network Security** | Private network with controlled exits | Only data exits explicitly; hack attempts blocked |

---

## **Who Can Access What?**

```
┌─────────────────────────────────────┐
│        Secure Central Database      │
│  (One Filing Cabinet Per Env)      │
├─────────────────────────────────────┤
│ 🟢 INTERNAL Data                    │  → Visible to BI Tools
├─────────────────────────────────────┤
│ 🟡 CONFIDENTIAL Data               │  → Visible to HR & Audit
├─────────────────────────────────────┤
│ 🔴 FINANCIAL Data                  │  → Visible to Finance
├─────────────────────────────────────┤
│ ⚫ AUDIT Data (Logs)                │  → Visible to Audit
└─────────────────────────────────────┘
```

---

## **What's Protected?**

### Employee Data
- Names, contact info, employment records
- Salary, benefits, performance reviews
- Medical information

### Financial Data
- Invoice & payment records
- Budget allocations
- Expense reports
- Contract terms

### Audit Data
- Who accessed what and when
- Changes to data
- Failed access attempts
- System alerts

---

## **Compliance & Regulations**

NAPTA meets requirements for:
- ✅ **GDPR** (General Data Protection Regulation) — EU data privacy
- ✅ **SOX** (Sarbanes-Oxley) — Financial transparency
- ✅ **HIPAA** (if applicable) — Healthcare data protection
- ✅ **Company Policy** — Internal data access rules

---

## **Current Status: What's Working?**

✅ **Strong Infrastructure**
- Secure network setup
- Encrypted storage
- Separate DEV/PROD environments
- Authentication system in place

✅ **Good Foundation**
- GitHub automation for deployments
- Centralized secrets management
- Private network architecture

---

## **Current Status: What Needs Work?**

⚠️ **Access Control** — Not fully enforced yet
- Users haven't been assigned to their data groups
- Action: Assign HR to see CONFIDENTIAL, Finance to see FINANCIAL, etc.
- Timeline: Q2 2026

⚠️ **Audit Logging** — Table exists but not recording yet
- We have the audit log system, but it's not "writing down" who accessed what
- Action: Activate logging in the database
- Timeline: Q2 2026

⚠️ **Missing Data Masking** — Sensitive info in analytics tools
- BI analysts might see salary numbers they shouldn't see
- Action: Hide sensitive fields in reporting tools
- Timeline: Q2 2026

---

## **Your Next Steps**

### **For HR:**
- [ ] Provide list of HR staff who need access
- [ ] Confirm data categories they need (yes/no to CONFIDENTIAL data)
- [ ] Set up single sign-on (SSO) for team

### **For Finance:**
- [ ] Provide list of Finance staff who need access
- [ ] Confirm they need FINANCIAL data access
- [ ] Verify compliance with SOX requirements

### **For Audit/Compliance:**
- [ ] Define audit logging retention period (how long to keep logs)
- [ ] Approve data classification scheme
- [ ] Set up SIEM (Security monitoring system)

---

## **Support & Questions**

- **Technical Questions?** → Contact DevOps Team
- **Access Issues?** → Contact Security Team
- **Compliance Questions?** → Contact Compliance Officer

---

**Document Version:** 1.0  
**Last Updated:** March 26, 2026  
**Next Review:** Q2 2026
