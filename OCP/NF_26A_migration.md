### pre-26A the SMF used a mix of:
```
CNLB (old style)
    |
Service Type=LoadBalancer
    |
Single Service
```

### 26A is moving towards a protocol-specific CLB architecture
```
CLBC-SBI   -> HTTP/HTTP2
CLBC-OAM   -> OAM TCP services
CLBC-TCP   -> CSDB/UDSF TCP services
CLBC-SCTP  -> Diameter SCTP
CLBC-UDP   -> PFCP/RADIUS/DNS/Charging UDP
CLBC-GTPC  -> GTP-C
```
### Before 26A

Many NFs were directly exposed through CNLB.
```
USM
 |
 | 172.23.145.19
 |
CNLB
 |
 +-- nf-netconf

USM
 |
 | 172.23.145.28
 |
CNLB
 |
 +-- smf-pfcpc

USM
 |
 | 172.23.145.22
 |
CNLB
 |
 +-- smf-radintf

Each service owned:

service:
type: LoadBalancer
```

### After 26A

The load balancing responsibility moves to dedicated CLBs.
Think of CLB as: Protocol Gateway

```
                 CLBC-OAM
             /      |      \
            /       |       \
      netconf    metric     UI


                 CLBC-TCP
                     |
                   UDSF


                 CLBC-SCTP
                     |
                 Diameter


                 CLBC-UDP
         /       /    \      \
      PFCP   DNS  Radius Offline


                 CLBC-GTPC
                     |
                    GTPC
```

#### Final Simplified 26A Architecture
```
                     External Networks
                              |
    ------------------------------------------------
    |              |            |          |        |
    |              |            |          |        |
 CLBC-OAM      CLBC-TCP    CLBC-SCTP  CLBC-UDP  CLBC-GTPC
    |              |            |          |        |
    |              |            |          |        |
    v              v            v          v        v

nf-netconf     nf-udsf      diameter    PFCP     GTPC
nf-metric                    SCTP       DNS
nf-resource                             Radius
nf-dbmanager                           Charging
nf-ui
```
#### OAM CLB
```
 cpNfClbcOam
  int0  -> 172.16.66.102
  ext0  -> 172.23.146.61
  ext0  -> 172.23.146.63

OAM Flow
                   USM
                    |
                    |
             172.23.146.61
                    |
              +-----------+
              | CLBC-OAM |
              +-----------+
                    |
            172.16.66.102
                    |
   ------------------------------------
   |         |        |      |        |
netconf   metric   UI   dbmanager resource

```
OAM Services Migrated
```
These are REMOVED from CNLB exposure:
  nf-netconf
  nf-metric
  nf-userinterface
  nf-dbmanager
  nf-resource
and moved behind:
  CLBC-OAM

```
#### TCP CLB cpNfClbcTcp
- CSDB communication
- UDFS communication
- Other TCP backend traffic
```
           CSDB
             |
             |
      172.23.146.76
             |
       +-----------+
       | CLBC-TCP |
       +-----------+
             |
      172.16.66.103
             |
             |
       nf-udsfintf
```

#### UDP CLB cpNfClbcUdp
```
              UDP Clients
                     |
                     |
      ----------------------------------
      |       |      |      |          |
    .109    .110   .111   .112      .113
      |       |      |      |          |
      +-------------------------------+
                      |
                 CLBC-UDP
                      |
                 172.16.67.105
                      |
      ----------------------------------
      |       |      |      |          |
   PFCP-C   DNS   PFCP-U  OFFLINE   RADIUS
```

#### SCTP CLB cpNfClbcSctp
 - Diameter
```
                  PCRF/HSS
                      |
                      |
                172.23.147.105
                      |
              +---------------+
              |  CLBC-SCTP    |
              +---------------+
                      |
                172.16.67.104
                      |
                 nf-diamintf
```


