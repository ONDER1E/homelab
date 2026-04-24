## Project: Homelab

This repository serves as the definitive source of truth for my homelab configuration, hardware topology, and service deployments. 

### Why?
The genesis of this documentation isn't just best practice - it's insurance. Following a dream where a bus... - related memory lapse led to the total neglect and eventual disposal of a perfectly good rack, I realized that my homelab exists mostly in my volatile memory.

**The goal of this repo is to ensure that if I ever forget anything, this README can guide me back to a functional stack.**

---

### Core Infrastructure

| Component | Hardware/Platform | Role | Status |
| :--- | :--- | :--- | :--- |
| **S1 (Custom built)** | Windows Server 2022 | VM, RM and storage backup | Online |
| **S2 (Lenovo P500)** | TrueNAS | Bulk data and micro services | Online |
| **ONT1 (Adtran SDX631)** | Proprietary | XGS-PON ONT | Online |
| **R1 (Technicolor FGA5330CFL)** | Proprietary (OpenWRT-based) | ISP Router | Online |
| **AP1 (Linksys Velop SPNMX56)** | Proprietary | Root Wi-Fi AP node | Online |
| **AP1 (Linksys Velop WHW03 V2)** | Proprietary | Wi-Fi AP node | Online |
| **SW1 (Cisco Nexus 3048TP)** | NX-OS | Shelf (ex-ToR switch) | **Offline** |
| **SW2 (ONT-S508CL-8S)** | Proprietary (V300SP10) | SFP+ (10G) Switch | Online |
| **SW3 (TL-SG105S)** | Proprietary | RJ45/Eth (1G) Switch | Online |

---

Notes:
- 10G capability (SW2) may seem unessessary/overkill, but the uplink's link speed is 3Gbps - this 10G setup is the most economically viable choice at the time of writing this without undersaturating the uplink.

- Cisco Nexus 3048TP is offline and decommissioned due being obsolete hardare, very loud and power inefficient. Currently acts as a shelf for Node 2 as it does not support a rack rail mounted configuration it's responsibility has been taken by signficantly lower power switches (SW2 and SW3).

- Prefer to keep R1 as main router instead of SW2 which is also L3 capable due to GUI simplicity and ISP friendly design.

- ONT1 is required (non-removable) as it is XGS-PON meaning it's unique Serial Number or a specific "Subscriber Line ID" is needed to authenticate with the IP to then obtain a valid uplink as well as it's FTTH wavelength specialty.

---

### Current look

![Image failed to render go to: https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/latest/20260410_033408.jpg](https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/latest/20260410_033408.jpg)