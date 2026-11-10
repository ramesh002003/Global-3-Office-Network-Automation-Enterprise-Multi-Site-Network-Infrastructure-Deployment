# Office A (HQ) — Full Fabric Network Diagram

## Layered Architecture

```mermaid
flowchart TB
    subgraph WAN["WAN / ISP Layer"]
        ISP1["ISP-1<br/>1G BGP"]
        ISP2["ISP-2<br/>1G BGP"]
    end

    subgraph SDWAN["SD-WAN Edge Layer"]
        WEA1["WAN-EDGE-A1<br/>C8500L-12H<br/>Color: 10"]
        WEA2["WAN-EDGE-A2<br/>C8500L-12H<br/>Color: 10"]
    end

    subgraph AGG["Aggregation Layer (VSS)"]
        AGG1["AGG-A1<br/>C9500-24Y<br/>VSS Member 1"]
        AGG2["AGG-A2<br/>C9500-24Y<br/>VSS Member 2"]
        AGG1 <-->|"VSS Link<br/>4×10G"| AGG2
    end

    subgraph CORE["Core Layer (VSS)"]
        CORE1["CORE-A1<br/>C9500-48Y<br/>STP Prio: 4096"]
        CORE2["CORE-A2<br/>C9500-48Y<br/>STP Prio: 4096"]
        CORE1 <-->|"VSS Link<br/>4×10G"| CORE2
    end

    subgraph DIST["Distribution Layer (VSS)"]
        DIST1["DIST-A1<br/>C9500-24Y<br/>STP Prio: 8192"]
        DIST2["DIST-A2<br/>C9500-24Y<br/>STP Prio: 8192"]
        DIST1 <-->|"VSS Link<br/>2×10G"| DIST2
    end

    subgraph ACCESS["Access Layer"]
        ACC1["ACCESS-A1<br/>C9300-48P<br/>PoE+ 48AP"]
        ACC2["ACCESS-A2<br/>C9300-48P<br/>PoE+ 48AP"]
    end

    subgraph AP["Wireless"]
        AP1["Wi-Fi 6 APs<br/>WPA3-Enterprise<br/>802.11r/w/k/v/u"]
    end

    subgraph SERVICES["Shared Services (VLAN 997)"]
        DNAC["DNA Center<br/>2× VM (HA)"]
        ISE1["ISE-1<br/>PAN+PSN+MnT"]
        ISE2["ISE-2<br/>PAN+PSN+MnT"]
        WLC1["WLC-A1<br/>C9800-40 (SSO)"]
        WLC2["WLC-A2<br/>C9800-40 (SSO)"]
        LOG["Graylog + OpenSearch<br/>Prometheus + Grafana<br/>Gitea + BIND9 + Chrony"]
    end

    ISP1 -->|"BGP"| WEA1
    ISP2 -->|"BGP"| WEA2
    WEA1 -->|"LACP 2×10G"| AGG1
    WEA2 -->|"LACP 2×10G"| AGG2
    AGG1 -->|"LACP 4×10G"| CORE1
    AGG2 -->|"LACP 4×10G"| CORE2
    CORE1 -->|"LACP 4×10G"| DIST1
    CORE2 -->|"LACP 4×10G"| DIST2
    DIST1 -->|"LACP 2×10G"| ACC1
    DIST2 -->|"LACP 2×10G"| ACC2
    ACC1 -->|"PoE+"| AP1
    ACC2 -->|"PoE+"| AP1

    CORE1 -.->|"Mgmt"| DNAC
    CORE1 -.->|"AAA"| ISE1
    CORE1 -.->|"AAA"| ISE2
    WLC1 <-->|"SSO Ring"| WLC2
    AP1 -.->|"CapWAP"| WLC1
    AP1 -.->|"CapWAP (backup)"| WLC2
```

## Redundancy Summary

| Layer | Redundancy Type | Failover Time |
|-------|----------------|---------------|
| WAN (ISP) | BGP Multihoming + ECMP | <1s (BGP convergence) |
| SD-WAN Edge | Dual vEdge, dual ISP | <1s (tunnel failover) |
| Aggregation | VSS (Virtual Switching System) | 0s (dual-active) |
| Core | VSS (Virtual Switching System) | 0s (dual-active) |
| Distribution | VSS (Virtual Switching System) | 0s (dual-active) |
| Access | Dual uplinks (LACP) | <50ms (LACP) |
| WLC | SSO (Stateful Switchover) | <1s |
| ISE | 2× PAN + PSN (full HA) | <2s |
| DNA Center | Active/Standby VM | <30s |
| Logging | Dual Graylog + Dual OpenSearch | 0s (active/standby) |

## VLAN & Subnet Map

```mermaid
flowchart LR
    subgraph VLANS["VLAN Assignments"]
        V10["VLAN 10<br/>CORP-USER<br/>10.100.0.0/16"]
        V20["VLAN 20<br/>CORP-VOICE<br/>10.101.0.0/16"]
        V30["VLAN 30<br/>CORP-SERVER<br/>10.102.0.0/16"]
        V40["VLAN 40<br/>CORP-IOT<br/>10.103.0.0/16"]
        V50["VLAN 50<br/>CORP-VIDEO<br/>10.104.0.0/16"]
        V997["VLAN 997<br/>MGMT<br/>10.1.0.0/24"]
        V998["VLAN 998<br/>NATIVE-TRUNK"]
        V999["VLAN 999<br/>PARKING-LOT"]
        V1000["VLAN 1000<br/>GUEST<br/>192.168.0.0/16"]
        V1001["VLAN 1001<br/>GUEST-WIFI"]
    end

    subgraph TRUSTSEC["TrustSec SGT Mapping"]
        SGT100["SGT 100: Corporate"]
        SGT200["SGT 200: Guest"]
        SGT300["SGT 300: IoT"]
        SGT400["SGT 400: Servers"]
        SGT500["SGT 500: Management"]
    end

    V10 --> SGT100
    V20 --> SGT100
    V30 --> SGT400
    V40 --> SGT300
    V50 --> SGT100
    V997 --> SGT500
    V1000 --> SGT200
    V1001 --> SGT200
```   
