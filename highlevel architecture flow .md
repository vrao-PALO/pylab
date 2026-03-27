flowchart TB

%% Identity
ENTRA["Entra ID (SSO RBAC MFA)"]
MI["Managed Identity"]

%% CI/CD
GH["GitHub Actions OIDC"]

%% External Sources
EXT1["External SaaS API"]
EXT2["SharePoint Graph"]
EXT3["MOM Data"]

%% DEV
subgraph DEV
DEVFUNC["Azure Functions"]
DEVDB1["Postgres INTERNAL"]
DEVDB2["Postgres CONFIDENTIAL"]
DEVDB3["Postgres FINANCIAL"]
DEVKV["Key Vault"]
DEVST["Storage"]
DEVNAT["NAT Gateway"]
end

%% PROD
subgraph PROD
PRODFUNC["Azure Functions"]
PRODDB1["Postgres INTERNAL"]
PRODDB2["Postgres CONFIDENTIAL"]
PRODDB3["Postgres FINANCIAL"]
PRODKV["Key Vault"]
PRODST["Storage"]
PRODNAT["NAT Gateway"]
end

%% Consumers
BI["BI Tools"]
HR["HR Users"]
FIN["Finance Users"]
AUD["Audit"]

%% Monitoring
MON["Monitor Logs"]
SIEM["SIEM"]

%% Flows
GH --> DEVFUNC
GH --> PRODFUNC

MI --> DEVFUNC
MI --> PRODFUNC

DEVFUNC --> DEVNAT
DEVNAT --> EXT1
DEVNAT --> EXT2
DEVNAT --> EXT3

PRODFUNC --> PRODNAT
PRODNAT --> EXT1
PRODNAT --> EXT2
PRODNAT --> EXT3

DEVFUNC --> DEVDB1
DEVFUNC --> DEVDB2
DEVFUNC --> DEVDB3
DEVFUNC --> DEVKV
DEVFUNC --> DEVST

PRODFUNC --> PRODDB1
PRODFUNC --> PRODDB2
PRODFUNC --> PRODDB3
PRODFUNC --> PRODKV
PRODFUNC --> PRODST

BI --> PRODDB1
HR --> PRODDB2
FIN --> PRODDB3
AUD --> PRODDB2

DEVFUNC --> MON
PRODFUNC --> MON
MON --> SIEM

ENTRA --> BI
ENTRA --> HR
ENTRA --> FIN
ENTRA --> AUD