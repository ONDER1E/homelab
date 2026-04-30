## Progress

### 27/04/2026
- bought a mellanox connectx3 sfp+ nic for £8.49 incl. VAT & shipping, at time of writing this usually sells for £15 excl. shipping so no complaints

![Image failed to render go to: https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/planning/Screenshot_20260427_015041_eBay.png](https://github.com/ONDER1E/homelab/blob/main/media_(image-video)/planning/Screenshot_20260427_015041_eBay.png)

- was confused why listing title only says sfp but the description says that the seller doesn't need it anymore as they no longer have 10-gigabit networking, also confirmed it was sfp+ via model number and experience that mellanox connectx3 cards are usually 10G

- I have a sfp+ passive dac lying around from upgrading S1 but the mellanox connectx4 card it has ( had to get cx4 as its the minimum version that supports windows as S1 runs windows datacentre 2022 currently unlike S2 which is truenas and has legacy diver support for cx3 ) didn't support that dac maybe because of eprom vendor or similar issues with the newer cx4 software


### 30/04/2026

- NIC arrived today, insalled as soon as I arrived from school (phone died so no pics)

- Didn't realised at first but the box was unusually small, realised that it might use a half-length PCIE bracket

- Checked image on ebay and realised it does look like it uses a half-length bracket

- Was wrapped in a Aldi bag in the box 😭 with a UniFI Dream Machine Special Edition manual attached to the box

- The box was probably reused and was an acessories box from that UniFi gateway

- The half-length bracket wasn't even screwed in, this made it easier to install since I'd have to remove it anyway so it would be physically compatible with the full-length PCIE bracket cage in S2

- Now the only thing holding that NIC in place is the friction fit of the PCIE DIMM and I'll take that risk

- installation was thus successful

- logged into TrueNAS on S2 and went to System > Network to setup link aggregation with the motherboard NIC ( 1G ) and the new Mellanox Cx3 NIC ( 10G ) in a FAILOVER config since LACP needs both NICs to be the same speed/duplex

- set the new NIC as primary due higher bandwidth & lower latency

- TrueNAS UI had me test the network config for 60s then I had to click to permanently apply it - like changing your display settings, very safe yet very annoying

- began conducting tests, first did a speedtest.net uplink test:

```
Linux truenas 6.12.33-production+truenas #1 SMP PREEMPT_DYNAMIC Mon Feb 23 17:38:27 UTC 2026 x86_64

        TrueNAS (c) 2009-2026, iXsystems, Inc. dba TrueNAS
        All rights reserved.
        TrueNAS code is released under the LGPLv3 and GPLv3 licenses with some
        source files copyrighted by (c) iXsystems, Inc. All other components
        are released under their own respective licenses.

        For more information, documentation, help or support, go here:
        http://truenas.com

Warning: the supported mechanisms for making configuration changes
are the TrueNAS WebUI, CLI, and API exclusively. ALL OTHERS ARE
NOT SUPPORTED AND WILL RESULT IN UNDEFINED BEHAVIOR AND MAY
RESULT IN SYSTEM FAILURE.

Welcome to TrueNAS
Last login: Thu Apr 30 15:13:23 BST 2026 on pts/3
truenas% speedtest

   Speedtest by Ookla

      Server: GSL Networks - London (id: 56811)
         ISP: Community Fibre Limited
Idle Latency:     1.49 ms   (jitter: 0.05ms, low: 1.41ms, high: 1.54ms)
    Download:  3152.31 Mbps (data used: 1.4 GB)                                                   
                  5.95 ms   (jitter: 1.06ms, low: 1.79ms, high: 11.22ms)
      Upload:  3157.14 Mbps (data used: 1.4 GB)                                                   
                  6.23 ms   (jitter: 0.40ms, low: 1.91ms, high: 7.01ms)
 Packet Loss:     0.0%
  Result URL: https://www.speedtest.net/result/c/aebcefe3-8712-426c-843d-926a0ec2c64b
truenas%
```

- Then did an iperf3 test with S1 to fully saturate the 10G link, here is the output from S1:

```
C:\Users\Administrator>iperf3 -c 192.168.1.206
Connecting to host 192.168.1.206, port 5201
[  5] local 192.168.1.10 port 64310 connected to 192.168.1.206 port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  1.11 GBytes  9.50 Gbits/sec
[  5]   1.00-2.00   sec  1.11 GBytes  9.49 Gbits/sec
[  5]   2.00-3.00   sec  1.11 GBytes  9.49 Gbits/sec
[  5]   3.00-4.00   sec  1.10 GBytes  9.49 Gbits/sec
[  5]   4.00-5.00   sec  1.11 GBytes  9.49 Gbits/sec
[  5]   5.00-6.00   sec  1.10 GBytes  9.49 Gbits/sec
[  5]   6.00-7.00   sec  1.11 GBytes  9.49 Gbits/sec
[  5]   7.00-8.00   sec  1.11 GBytes  9.49 Gbits/sec
[  5]   8.00-9.00   sec  1.10 GBytes  9.49 Gbits/sec
[  5]   9.00-10.00  sec  1.10 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.00  sec  11.1 GBytes  9.49 Gbits/sec                  sender
[  5]   0.00-10.00  sec  11.1 GBytes  9.49 Gbits/sec                  receiver

iperf Done.

C:\Users\Administrator>
```

- also conducted another iperf3 test with multiple streams to test stability:

```
C:\Users\Administrator>iperf3 -c 192.168.1.206 -P 8
Connecting to host 192.168.1.206, port 5201
[  5] local 192.168.1.10 port 64561 connected to 192.168.1.206 port 5201
[  7] local 192.168.1.10 port 64562 connected to 192.168.1.206 port 5201
[  9] local 192.168.1.10 port 64563 connected to 192.168.1.206 port 5201
[ 11] local 192.168.1.10 port 64564 connected to 192.168.1.206 port 5201
[ 13] local 192.168.1.10 port 64565 connected to 192.168.1.206 port 5201
[ 15] local 192.168.1.10 port 64566 connected to 192.168.1.206 port 5201
[ 17] local 192.168.1.10 port 64567 connected to 192.168.1.206 port 5201
[ 19] local 192.168.1.10 port 64568 connected to 192.168.1.206 port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec   164 MBytes  1.37 Gbits/sec
[  7]   0.00-1.00   sec   166 MBytes  1.39 Gbits/sec
[  9]   0.00-1.00   sec   164 MBytes  1.38 Gbits/sec
[ 11]   0.00-1.00   sec   164 MBytes  1.37 Gbits/sec
[ 13]   0.00-1.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   0.00-1.00   sec   164 MBytes  1.37 Gbits/sec
[ 17]   0.00-1.00   sec  78.4 MBytes   657 Mbits/sec
[ 19]   0.00-1.00   sec  87.5 MBytes   734 Mbits/sec
[SUM]   0.00-1.00   sec  1.12 GBytes  9.63 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   1.00-2.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   1.00-2.00   sec  75.9 MBytes   637 Mbits/sec
[ 19]   1.00-2.00   sec  85.6 MBytes   718 Mbits/sec
[SUM]   1.00-2.00   sec  1.10 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   2.00-3.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   2.00-3.00   sec  76.2 MBytes   639 Mbits/sec
[ 19]   2.00-3.00   sec  85.6 MBytes   718 Mbits/sec
[SUM]   2.00-3.00   sec  1.11 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   3.00-4.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   3.00-4.00   sec  75.9 MBytes   636 Mbits/sec
[ 19]   3.00-4.00   sec  85.9 MBytes   720 Mbits/sec
[SUM]   3.00-4.00   sec  1.11 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   4.00-5.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   4.00-5.00   sec  75.8 MBytes   636 Mbits/sec
[ 19]   4.00-5.00   sec  85.8 MBytes   719 Mbits/sec
[SUM]   4.00-5.00   sec  1.10 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   5.00-6.00   sec  76.1 MBytes   639 Mbits/sec
[ 19]   5.00-6.00   sec  85.8 MBytes   719 Mbits/sec
[SUM]   5.00-6.00   sec  1.11 GBytes  9.50 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   6.00-7.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   6.00-7.00   sec  75.9 MBytes   636 Mbits/sec
[ 19]   6.00-7.00   sec  85.8 MBytes   719 Mbits/sec
[SUM]   6.00-7.00   sec  1.11 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   7.00-8.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   7.00-8.00   sec  76.0 MBytes   638 Mbits/sec
[ 19]   7.00-8.00   sec  85.5 MBytes   717 Mbits/sec
[SUM]   7.00-8.00   sec  1.10 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[  7]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[  9]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[ 11]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[ 13]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[ 15]   8.00-9.00   sec   162 MBytes  1.36 Gbits/sec
[ 17]   8.00-9.00   sec  75.9 MBytes   636 Mbits/sec
[ 19]   8.00-9.00   sec  85.9 MBytes   720 Mbits/sec
[SUM]   8.00-9.00   sec  1.11 GBytes  9.50 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   9.00-10.00  sec   162 MBytes  1.36 Gbits/sec
[  7]   9.00-10.00  sec   162 MBytes  1.36 Gbits/sec
[  9]   9.00-10.00  sec   161 MBytes  1.35 Gbits/sec
[ 11]   9.00-10.00  sec   162 MBytes  1.36 Gbits/sec
[ 13]   9.00-10.00  sec   162 MBytes  1.36 Gbits/sec
[ 15]   9.00-10.00  sec   162 MBytes  1.36 Gbits/sec
[ 17]   9.00-10.00  sec  76.5 MBytes   642 Mbits/sec
[ 19]   9.00-10.00  sec  85.5 MBytes   717 Mbits/sec
[SUM]   9.00-10.00  sec  1.11 GBytes  9.49 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[  5]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  receiver
[  7]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[  7]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  receiver
[  9]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[  9]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  receiver
[ 11]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[ 11]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  receiver
[ 13]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[ 13]   0.00-10.00  sec  1.58 GBytes  1.35 Gbits/sec                  receiver
[ 15]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  sender
[ 15]   0.00-10.00  sec  1.58 GBytes  1.36 Gbits/sec                  receiver
[ 17]   0.00-10.00  sec   762 MBytes   640 Mbits/sec                  sender
[ 17]   0.00-10.00  sec   761 MBytes   638 Mbits/sec                  receiver
[ 19]   0.00-10.00  sec   859 MBytes   720 Mbits/sec                  sender
[ 19]   0.00-10.00  sec   857 MBytes   719 Mbits/sec                  receiver
[SUM]   0.00-10.00  sec  11.1 GBytes  9.51 Gbits/sec                  sender
[SUM]   0.00-10.00  sec  11.1 GBytes  9.49 Gbits/sec                  receiver

iperf Done.

C:\Users\Administrator>
```

### Conclusion

- Upgrade was successful though only major issue is the way the NIC is mounted
- tests show S2 now has full ability to saturate the uplink unlike the motherboards NIC with is 1G
- can now also do backup/data transfers at 10G between S1 and S2
- SW2 with 1 uplink now has gone from 1 to 2 downlinks