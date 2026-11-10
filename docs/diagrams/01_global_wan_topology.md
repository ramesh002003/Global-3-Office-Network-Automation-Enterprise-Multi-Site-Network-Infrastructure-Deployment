# Global WAN Topology — SD-WAN Fabric

## Overview

```mermaid
flowchart TB
    subgraph INTERNET["🌐 Internet"]
        ISP1["ISP-1 (1G BGP)<br/>AS 65002"]
        ISP2["ISP-2 (1G BGP)<br/>AS 65003"]
        ISP3["ISP-1 (500M)<br/>HSRP"]
        ISP4["ISP-2 (500M)<br/>HSRP"]
        ISP5["ISP-1 (100M)"]
        ISP6["4G Backup"]
    end

    subgraph SITE_A["🏢 OFFICE A — HQ (500+ users)"]
        WEA1["WAN-EDGE-A1<br/>C8500L-12H"]
        WEA2["WAN-EDGE-A2<br/>C8500L-12H"]
    end

    subgraph SITE_B["🏢 OFFICE B — Regional (150-300 users)"]
        WEB1["WAN-EDGE-B1<br/>C8500L-12H"]
        WEB2["WAN-EDGE-B2<br/>C8500L-12H"]
    end

    subgraph SITE_C["🏢 OFFICE C — Branch (20-50 users)"]
        WEC1["WAN-EDGE-C1<br/>C8300-1N-12G"]
    end

    ISP1 -->|"BGP 1G"| WEA1
    ISP2 -->|"BGP 1G"| WEA2
    ISP3 -->|"HSRP 500M"| WEB1
    ISP4 -->|"HSRP 500M"| WEB2
    ISP5 -->|"Static 100M"| WEC1
    ISP6 -->|"4G Failover"| WEC1

    WEA1 <-->|"IPsec 1G<br/>Tunnel A↔B"| WEB1
    WEA1 <-->|"IPsec 1G"| WEB2
    WEA2 <-->|"IPsec 1G"| WEB1
    WEA2 <-->|"IPsec 1G"| WEB2
    WEA1 <-->|"IPsec 100M<br/>Tunnel A↔C"| WEC1
    WEB1 <-->|"IPsec 100M<br/>Tunnel B↔C"| WEC1
    WEB2 <-->|"IPsec 100M"| WEC1

    subgraph CONTROL["⚙️ SD-WAN Control Plane (Office A)"]
        VM1["vManage-1<br/>(Primary)"]
        VM2["vManage-2<br/>(Secondary)"]
        VS1["vSmart-1"]
        VS2["vSmart-2"]
        VB1["vBond-1"]
        VB2["vBond-2"]
    end

    WEA1 -.->|"DTLS"| VM1
    WEA2 -.->|"DTLS"| VM1
    WEB1 -.->|"DTLS"| VM1
    WEB2 -.->|"DTLS"| VM1
    WEC1 -.->|"DTLS"| VM1
    WEA1 -.->|"BFD"| VB1
    WEB1 -.->|"BFD"| VB1
    WEC1 -.->|"BFD"| VB1
```

## Tunnel Matrix

| From | To | Bandwidth | Encryption | Purpose |
|------|-----|-----------|------------|---------|
| A1 | B1 | 1 Gbps | AES-256-GCM | Primary inter-site |
| A1 | B2 | 1 Gbps | AES-256-GCM | Redundant inter-site |
| A2 | B1 | 1 Gbps | AES-256-GCM | Redundant inter-site |
| A2 | B2 | 1 Gbps | AES-256-GCM | Primary inter-site |
| A1 | C1 | 100 Mbps | AES-256-GCM | HQ to Branch |
| B1 | C1 | 100 Mbps | AES-256-GCM | Regional to Branch (backup) |
| B2 | C1 | 100 Mbps | AES-256-GCM | Regional to Branch (backup) |

## App-Aware Routing Policy

```mermaid
flowchart LR
    APP["Application Traffic"]

    APP -->|"VoIP / Video"| LOW["Lowest Latency Path<br/>(SD-WAN QoS DSCP EF)"]
    APP -->|"SaaS (O365, Zoom)"| DIB["Direct Internet Breakout<br/>(Local Edge)"]
    APP -->|"Data Center"| HQ["Via HQ (Office A)<br/>(Centralized)"]
    APP -->|"Inter-site"| MESH["Encrypted Overlay<br/>(Full Mesh)"]
    APP -->|"Guest / IoT"| ISOL["Isolated VLAN<br/>(SGT-based)"]
```   
