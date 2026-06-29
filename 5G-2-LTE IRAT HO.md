```
 UE          Source gNB       Target eNB         AMF             MME             SGW           SMF/UPF

 |               |               |                |               |               |               |
 |======= 1. Active 5G User Plane Session (PDU Sessions via UPF) =================================|
 |               |               |                |               |               |               |
 |--2.MeasReport>|               |                |               |               |               |
 |               |--3. Handover Required -------->|               |               |               |
 |               |  (NGAP: Handover Required)     |               |               |               |
 |               |               |                |--4. Forward Relocation Req--->|               |
 |               |               |                |     (GTPv2-C: IMSI, MM/SM Context, SMF F-TEID)|
 |               |               |                |               |--5. Create Indirect Tunnel Req|
 |               |               |                |               |     (If Indirect Forwarding)  |
 |               |               |                |               |<--6. Create Indirect Tunnel Res
 |               |               |<--7. Handover Request ---------|               |               |
 |               |               |  (S1AP: Handover Request)      |               |               |
 |               |               |--8. Handover Request Ack ------>|               |               |
 |               |               |  (S1AP: Handover Request Ack)  |               |               |
 |               |               |                |               |--9. Create Indirect Tunnel Req|
 |               |               |                |               |     (Target eNB TEIDs info)   |
 |               |               |                |               |<--10.Create Indirect Tunnel Res
 |               |               |<--11. Forward Relocation Resp -|               |               |
 |               |               |     (GTPv2-C: SGW/eNB Forwarding TEIDs)        |               |
 |               |<--12. Handover Command --------|               |               |               |
 |               |  (NGAP: Handover Command)      |               |               |               |
 |               |               |                |               |               |               |
 |               |====== 13. Data Forwarding Tunnel (Source gNB -> SGW -> Target eNB) ===========>|
 |               |               |                |               |               |               |
 |<--14. RRC Reconfig ------------|                |               |               |               |
 |   (MobilityFromNRCommand)     |                |               |               |               |
 |               |               |                |               |               |               |
 |--15. RACH Preambles---------->|                |               |               |               |
 |<-16. RACH Response-----------|                |               |               |               |
 |--17. RRC Conn Reconfig Comp ->|                |               |               |               |
 |               |               |--18. Handover Notify --------->|               |               |
 |               |               |  (S1AP: Handover Notify)       |               |               |
 |               |               |                |--19. Forward Relocation Complete------------->|
 |               |               |                |<--20. Forward Relocation Complete Ack --------|
 |               |               |                |               |               |               |
 |               |               |                |               |--21. Modify Bearer Request -->|
 |               |               |                |               | (GTPv2-C: eNB F-TEID for DL)  |
 |               |               |                |               |               |--22. Modify -->|
 |               |               |                |               |               |  Bearer Req    |
 |               |               |                |               |               |<--23. Modify --|
 |               |               |                |               |               |  Bearer Resp   |
 |               |               |                |               |<--24. Modify Bearer Response -|
 |               |               |                |               |     (GTPv2-C: Charging/PAA)   |
 |               |               |                |               |               |               |
 |======= 25. Active 4G LTE User Plane Session (E-RABs via SGW/UPF) ==============================|
 |               |               |                |               |               |               |
 |               |               |                |--26. Delete Indirect Tunnel Req-------------->|
 |               |               |                |<--27. Delete Indirect Tunnel Resp ------------|
 |               |               |                |               |               |               |
 |               |               |                |--28. UE Context Release Command ------------->|
 |               |<--29. UE Context Release ------|               |               |               |
 |               |  (NGAP: UE Context Release Cmd)|               |               |               |
 |               |--30. UE Context Release Comp ->|               |               |               |
 |               |  (NGAP: UE Context Release Com)|               |               |               |

```
