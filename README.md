## Project: Homelab

This repository serves as the definitive source of truth for my homelab configuration, hardware topology, and service deployments. 

### Recent Changes (newest first)

- [Networking Cable Management](https://github.com/ONDER1E/homelab/blob/main/logs/04-05-26_networking_cable_management.md)

- [Physical reorder](https://github.com/ONDER1E/homelab/blob/main/logs/01-05-2026_configuration_change.md)

- [S2: Upgraded primary NIC to 10G](https://github.com/ONDER1E/homelab/blob/main/logs/27-4-2026_S2_networking_upgrade.md)

- [Moved houses](https://github.com/ONDER1E/homelab/blob/main/logs/01-10-2026_lift_and_shift.md)


### Why?
The genesis of this documentation isn't just best practice - it's insurance. Following a dream where a bus... - related memory lapse led to the total neglect and eventual disposal of a perfectly good rack, I realized that my homelab exists mostly in my volatile memory.

**The goal of this repo is to ensure that if I ever forget anything, this README can guide me back to a functional stack.**

---

### Core Infrastructure

#### Active and online

| Component | Hardware/Platform | Role |
| :--- | :--- | :--- |
| **S1 (Custom built)** | Windows Server 2022 | VM, RM and storage backup |
| **S2 (Lenovo P500)** | TrueNAS | Bulk data and micro services |
| **ONT2 (Adtran SDX631q)** | Proprietary | ISP XGS-PON ONT |
| **GW1 (Linksys Velop SPNM60)** | Proprietary | ISP Gateway |
| **SW2 (ONT-S508CL-8S)** | Proprietary (V300SP10) | SFP+ (10G) Switch |
| **SW3 (TL-SG105S)** | Proprietary | RJ45/Eth (1G) Switch |

#### Offline, deactivated or removed

| Component | Hardware/Platform | Role |
| :--- | :--- | :--- |
| **SW1 (Cisco Nexus 3048TP)** | NX-OS | Shelf (ex-ToR switch) |
| **ONT1 (Adtran SDX631)** | Proprietary | ISP XGS-PON ONT |
| **R1 (Technicolor FGA5330CFL)** | Proprietary (OpenWRT-based) | ISP Router |
| **AP1 (Linksys Velop SPNMX56)** | Proprietary | Root Wi-Fi AP node |
| **AP2 (Linksys Velop WHW03 V2)** | Proprietary | Wi-Fi AP node |
---

### Notes

- 10G capability (SW2) may seem unessessary/overkill, but the uplink's link speed is 3Gbps - this 10G setup is the most economically viable choice at the time of writing this without undersaturating the uplink.

- Cisco Nexus 3048TP is offline and decommissioned due being obsolete hardare, very loud and power inefficient. Currently acts as a shelf for S2 as it does not support a rack rail mounted configuration it's responsibility has been taken by signficantly lower power switches (SW2 and SW3).

- After moving houses I had to deactivate R1, AP1 and AP2 because in doing so, I had to renew my ISP contract, the 3Gbps uplink I had no longer existed meaning I could only downgrade to the 2.5G plan, the ISP then gave my a new ONT (ONT2) which only has a 2.5G downlink compared to the 10G downlink ONT1 has, this makes it incompatible with R1 which only accepts 10G uplinks meaning that GW1 has replace R1, AP1 and AP2.

- Following extensive speed, coverage/range and reliabilty tests, it was concluded that GW1 is a sufficient replacement with the added benefit of lower power consumption (less AC-DC conversions to single units)

- GW1 position is to be temporary (which is why changes in the network diagram below are displayed rather than updated) but implementation to replace it is expected to be delayed due to inherently being a low prioriy task
---

### Networking at a glance

<pre>
                      ┌─────────────────────────┐
                      │    [ ISP / Internet ]   │
                      └────────────┬────────────┘
                                   │ (XGS-PON Fiber) SC/APC SM OS2
                      ┌────────────▼────────────┐
                      │        ONT <del>1</del> 2          │  <del>Adtran SDX631</del> Adtran SDX631q
                      └────────────┬────────────┘
                                   │ (Ethernet WAN) RJ45 CAT8
                      ┌────────────▼────────────┐
                      │   <del>Router 1</del> Gateway 1    │  <del>Technicolor FGA5330CFL</del> Linksys Velop SPNM60
                      └──────┬────────────┬─────┘  <del>(Main Router)</del>
                             │            │
                             │            │ RJ45 CAT8 ──► SFP+ RJ45 Transceiver
                             │ RJ45 CAT8  │
        ┌────────────────────▼───┐    ┌───▼────────────────────┐
        │        Switch 3        │    │        Switch 2        │  ONT-S508CL-8S
        └─┬──┬──┬──┬─────────────┘    └─────────────┬──┬───────┘  (10G SFP+ Switch)
          │  │  │  │   TP-Link TL-SG105S            │  │
          │  │  │  │   (1G RJ45 Switch)             │  │
          │  │  │  │                                │  │
          │  │  │  │   (RJ45)      ┌────────────┐   │  │  (SFP+ Passive DAC * 2)
          │  │  │  └──────────────►│  SERVER 1  │◄──┘  │
          │  │  │                  └────────────┘      │
          │  │  │                (LAGP in Failover)    │
          │  │  │                                      │
          │  │  │      (RJ45)      ┌────────────┐      │
          │  │  └─────────────────►│  SERVER 2  │◄─────┘
          │  │                     └────────────┘
          │  │                   (LAGP in Failover)
          │  └────────────────────────────────────────────────────────────┐
          │                                                               │
          ▼ <del>(RJ45)</del>                                                        ▼ <del>(RJ45)</del>
┌────────────────────────────────────────┐                     ┌──────────────────┐
│             <del>(Mains Wiring)</del>             │                     │  <del>Access Point 1</del>  │ <del>Linksys Velop SPNMX56</del>
│   <del>Powerline ─────────────► Powerline</del>   │                     │                  │ <del>(Root Wi-Fi Node)</del>
└───────┬────────────────────────────────┘                     └──────────────────┘
        │ <del>(RJ45)</del>
┌───────▼────────┐
│ <del>Access Point 2</del> │  <del>Linksys Velop WHW03</del>
└────────────────┘  <del>(Wi-Fi Node)</del>
</pre>

 

---

### Current look

![Image failed to render go to: https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/latest/20261001_175943.jpg](https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/latest/20261001_175943.jpg)