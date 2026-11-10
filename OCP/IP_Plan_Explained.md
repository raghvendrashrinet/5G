### Internal Pool (int0)
This is the CLB-to-backend network.
```
  VLAN: 132
  Network: 10.0.132.0/22
 
  Usable internal VIP space:
   10.0.133.x
```

 Uses:
  - CLBC int0 IPs
  - CLBC internal route IP
  - Backend pod NAD IPs
```
CLBC-SBI
10.0.133.70-73

SBI route
10.0.133.94

CLBC-GTPC
10.0.133.66-69

GTPC route
10.0.133.90

PDUSESS
10.0.133.74-77

NFMCLIENT
10.0.133.82-85
```

### External Pool (ext0)
This is where VIPs are presented to peers.
```
VLAN 150
10.0.150.0/24
```
Example
```
SBI VIP
10.0.150.61

CLBC pod external IPs
10.0.150.102
10.0.150.103


For GTPC:

VIP
10.0.150.132
10.0.150.135

CLBC external IPs
10.0.150.133
10.0.150.134

