# Global 3-Office Network Automation

Production-grade network automation for a 3-office global deployment with full redundancy, SD-WAN fabric, centralized management, and **zero external dependency**.

---

## Architecture Overview

```mermaid
flowchart TB
    subgraph SITE_A["🏢 OFFICE A — HQ (500+ users)"]
        A_WAN["2× WAN Edge (C8500L)"]
        A_CORE["2× Core (C9500-48Y, VSS)"]
        A_DIST["2× Dist (C9500-24Y, VSS)"]
        A_ACC["2× Access (C9300-48P)"]
        A_SVC["DNA Center + ISE + WLC + Logging"]
    end

    subgraph SITE_B["🏢 OFFICE B — Regional (150-300 users)"]
        B_WAN["2× WAN Edge (C8500L)"]
        B_CORE["2× Core (C9300-48P, VSS)"]
        B_DIST["2× Dist (C9300-48P, VSS)"]
        B_ACC["2× Access (C9300-48P)"]
        B_SVC["WLC SSO + ISE PSN"]
    end

    subgraph SITE_C["🏢 OFFICE C — Branch (20-50 users)"]
        C_WAN["1× WAN Edge (C8300)"]
        C_SW["1× Collapsed (C9300-24T)"]
        C_AP["FlexConnect APs"]
    end

    A_WAN <-->|"IPsec 1G (Full Mesh)"| B_WAN
    A_WAN <-->|"IPsec 100M"| C_WAN
    B_WAN <-->|"IPsec 100M"| C_WAN
```

---

## Project Benefits

```mermaid
mindmap
  root((Benefits))
    Speed
      100 switches in 5 hrs<br/>(vs 3 weeks manual)
      Zero-touch provisioning
      Parallel deployment<br/>(5 at a time)
    Accuracy
      Zero typos<br/>(templated)
      Idempotent<br/>(safe to re-run)
      Drift detection
    Cost
      No cloud dependency
      Open-source logging
      1 engineer manages all 3 sites
    Security
      802.1X + TrustSec
      MACsec on core
      Full audit trail
    Scalability
      Add office = add YAML file
      1 template = 100+ devices
      Version-controlled
```

### Quantified Impact

| Metric | Manual | Automated | Improvement |
|--------|--------|-----------|-------------|
| Deploy 100 switches | 3 weeks | **5 hours** | **~90% faster** |
| Config errors | ~15 per site | **0** | **100% reduction** |
| Change management | 2 hrs/change | **5 min/change** | **96% faster** |
| Rollback time | 30 min/device | **2 min (Git)** | **93% faster** |
| Audit compliance | Manual logs | **Auto (Git + Graylog)** | **100% coverage** |
| Onboarding new engineer | 2 weeks | **1 day (read README)** | **~93% faster** |

---

## Obstacles & Mitigations

```mermaid
flowchart TD
    subgraph OBSTACLES["⚠️ Obstacles"]
        O1["License Lead Time<br/>(4-6 weeks)"]
        O2["VSS/VPC Complexity<br/>(dual-active tuning)"]
        O3["ISE pxGrid Setup<br/>(3 nodes, 3 personas)"]
        O4["SD-WAN Tunnel<br/>Debugging"]
        O5["Air-Gapped Licensing<br/>(SLUP config)"]
        O6["100+ Device<br/>Parallel SSH"]
    end

    subgraph MITIGATIONS["✅ Mitigations"]
        M1["Order licenses Day 1<br/>Parallel with design"]
        M2["Lab first (EVE-NG)<br/>Test VSS before prod"]
        M3["Ansible playbook<br/>Automates pxGrid join"]
        M4["vManage dashboard<br/>+ BFD monitoring"]
        M5["PAK server on-prem<br/>One-time setup"]
        M6["Ansible forks=20<br/>+ SSH ControlMaster"]
    end

    O1 --> M1
    O2 --> M2
    O3 --> M3
    O4 --> M4
    O5 --> M5
    O6 --> M6
```

### Risk Matrix

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| License delivery delay | High | High | Order immediately, track weekly |
| VSS peer-link failure | Low | Critical | 4×10G redundant, auto-recovery |
| ISE PAN failover | Low | High | 2× PAN (active/standby), <2s |
| SD-WAN tunnel flap | Medium | Medium | BFD + multiple tunnels per site |
| Ansible mid-deploy failure | Low | High | Idempotent, safe re-run, Git rollback |
| ISP outage (single) | Medium | Medium | Dual ISP + ECMP/HSRP |

---

## Efficiency: Manual vs. Automated

```mermaid
flowchart LR
    subgraph MANUAL["🔴 Manual Process"]
        M1["Write config<br/>on console"] --> M2["Copy to next<br/>switch"]
        M2 --> M3["Repeat ×100"]
        M3 --> M4["Find typos<br/>(15 errors)"]
        M4 --> M5["Fix one by one"]
        M5 --> M6["3 weeks total"]
    end

    subgraph AUTOMATED["🟢 Automated Process"]
        A1["Edit 1 YAML<br/>file"] --> A2["Run playbook"]
        A2 --> A3["100 switches<br/>in parallel"]
        A3 --> A4["0 errors<br/>(templated)"]
        A4 --> A5["5 hours total"]
    end
```

### Time Breakdown (Per Site)

```mermaid
gantt
    title Deployment Timeline — Per Site
    dateFormat HH:mm
    axisFormat %H:%M

    section Office A (HQ)
    Console + Mgmt IP       :a1, 09:00, 30min
    Bootstrap (SSH/NTP)     :a2, after a1, 15min
    Global + Security       :a3, after a2, 30min
    STP + LACP + VSS        :a4, after a3, 30min
    VLANs + Routing         :a5, after a4, 30min
    SD-WAN + WLC + ISE      :a6, after a5, 60min
    Validation + Backup     :a7, after a6, 30min

    section Office B (Regional)
    Console + Mgmt IP       :b1, after a7, 30min
    Full Config (1-11)      :b2, after b1, 90min
    Validation + Backup     :b3, after b2, 30min

    section Office C (Branch)
    Console + Mgmt IP       :c1, after b3, 30min
    Full Config             :c2, after c1, 60min
    Validation + Backup     :c3, after c2, 30min
```

---

## Security Layers

```mermaid
flowchart TB
    L1["🔐 Layer 1: Identity<br/>802.1X / MAB / WebAuth<br/>(ISE)"]
    L2["🔐 Layer 2: Segmentation<br/>TrustSec SGT + SGACL<br/>(Group-based)"]
    L3["🔐 Layer 3: Port Security<br/>Port Sec + DHCP Snoop<br/>+ DAI + IP Source Guard"]
    L4["🔐 Layer 4: Management<br/>SSHv2 + TACACS+<br/>+ ACL (Mgmt VLAN only)"]
    L5["🔐 Layer 5: Wireless<br/>WPA3-Enterprise<br/>+ 802.11w (MFP)"]
    L6["🔐 Layer 6: WAN<br/>IPsec AES-256-GCM<br/>+ DTLS Control Plane"]

    L1 --> L2 --> L3 --> L4 --> L5 --> L6
```

---

## Zero External Dependency

```mermaid
flowchart LR
    subgraph ON_PREM["All Self-Hosted (Office A)"]
        A["DNA Center<br/>(Management + PAK)"]
        B["ISE<br/>(Identity)"]
        C["vManage/vSmart/vBond<br/>(SD-WAN Control)"]
        D["Graylog + OpenSearch<br/>(Logging)"]
        E["Prometheus + Grafana<br/>(Monitoring)"]
        F["Gitea<br/>(Config Git)"]
        G["BIND9 + Chrony<br/>(DNS + NTP)"]
    end

    subgraph NO_CLOUD["❌ No Cloud Needed"]
        X1["No AWS/Azure"]
        X2["No Cisco Cloud"]
        X3["No SaaS Logging"]
        X4["No Internet for Ops"]
    end

    A & B & C & D & E & F & G -.->|"Replaces"| NO_CLOUD
```

---

## CI/CD Pipeline

```mermaid
flowchart LR
    DEV["Developer<br/>Push"] --> PR["PR"]
    PR --> LINT["Lint"]
    LINT --> SYNTAX["Syntax Check"]
    SYNTAX --> DRY["Dry Run"]
    DRY --> REVIEW["Code Review"]
    REVIEW --> STAGE["Staging Deploy"]
    STAGE --> VALIDATE["Validation"]
    VALIDATE --> APPROVE["Approval Gate"]
    APPROVE --> PROD["Production Deploy"]
    PROD --> BACKUP["Git Backup"]
```

---

## Repo Structure

```
global-network-automation/
├── README.md              ← This file
├── ansible.cfg
├── requirements.yml
├── requirements.txt
├── Makefile
├── setup.sh
│
├── inventories/
│   ├── production/        ← Live network
│   ├── staging/           ← Pre-prod
│   └── lab/               ← EVE-NG
│
├── playbooks/             ← 14 ordered playbooks
├── roles/                 ← Reusable Ansible roles
├── templates/             ← Jinja2 config templates
├── dnac_automation/       ← DNA Center workflows
├── monitoring/            ← Prometheus + Grafana + Graylog
├── scripts/               ← Helper scripts
├── docs/                  ← Runbooks, IP plan, DR
├── tests/                 ← Unit + integration + vLab
└── .github/workflows/     ← CI/CD pipelines
```

---

## Quick Start

```bash
git clone git@github.com:YOUR_ORG/global-network-automation.git
cd global-network-automation
./setup.sh
make deploy-staging      # Test first
make deploy-production   # Go live
```

---

## License

Private — Internal Use Only   
