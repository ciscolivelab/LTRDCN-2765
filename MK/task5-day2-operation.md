#Task 5: Day 2 operation using Ansible

In this section, we will use automation to perform following day 2 operation tasks.

- Backup running configurations on all leaf and spine switches
- Verify underlay ospf, bgp and pim neighbors
- Verify overlay nve peer, host route, bgp update
- Baseline configuration comparison
- Add new VNIs into the existing fabric

---
### Step 1: Backup running configurations
In this section, you will use **ios_config** module to backup running configuration on each switch, the backup file will be saved to a local **“backup”** folder.  The backup argument create a full backup of the current running-config of each switch.  The backup file is written to the backup folder in the playbook root directory. If the directory does not exist, it is created.

* On **Atom**, open up the project folder **“EVPN-Ansible”** and create new file under `EVPN-Ansible`. Name the new file `get_config.yml`.

* **Click** `File` and `Save` . This will save the playbook, and also ftp the playbook to Ansible server using pre-configured “**remote-sync**” package.

~~~YAML
---
   - hosts: spine,leaf,jinja2_leaf,jinja2_spine
     tasks:
      - name: save running
        cisco.nxos.nxos_config:
          backup: yes
~~~

- On the Ansible node (using MTputty via SSH), run **“get_config.yml”** playbook and verify the backup configurations in **“backup”** folder by using below commands:

		[root@rhel7-tools EVPN-Ansible]# ansible-playbook get_config.yml
		[root@rhel7-tools EVPN-Ansible]# ls -lrt
		[root@rhel7-tools EVPN-Ansible]# ls backup


	You may further view the contents of the files under backup folder by using cat, less or more commands.  Below screenshot shows the output of above commands

	![](pic/5-1-1.png)

---
### Step 2:	Verify underlay and overlay

In this step, you will verify underlay and overlay operation using ansible playbook. The playbook will be applied to all leaf switches to verify the below commands:

***Underlay***

	-	show ip ospf neighbor
	-	show ip bgp sum
	-	show ip pim neighbor
***Overlay***

	-	show nve vni
	-	show nve peer
	-	show ip route vrf Tenant-1
	-	show bgp l2vpn evpn
	-	show l2route evpn mac-ip all


*  Switch to “Atom”, **right click** on the folder `EVPN-Ansible` and create a new playbook named `verify_fabric.yml`. Enter this file name and hit enter.

```YAML
---
  - hosts: leaf, jinja2_leaf
    connection: local
    gather_facts: false
    tasks:
      - name: verify underlay
        register: underlay_output
        cisco.nxos.nxos_command:
          commands:
            - show ip ospf neighbors
            - show ip bgp sum
            - show ip pim neighbor
        tags: underlay
      - debug: var=underlay_output.stdout_lines
        tags: underlay
      - name: Verify Overlay
        register: overlay_output
        cisco.nxos.nxos_command:
          commands:
            - show nve vni
            - show nve peer
            - show ip route vrf Tenant-1
            - show bgp l2vpn evpn
            - show l2route evpn mac-ip all
        tags: overlay
      - debug: var=overlay_output.stdout_lines
        tags: overlay
```

* **Click** `File` and `Save` . This will save the playbook, and also ftp the playbook to Ansible server using pre-configured “**remote-sync**” package.

* On the Ansible node (via MTPutty), run **verify_fabric.yml** playbook and verify the output for **underlay** by executing below command (using respective tag):
~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook verify_fabric.yml --tags "underlay"
~~~

***
<font style=background-color:yellow>The output shows ospf, bgp and pim neighbors for all leaf switches</font>
***

* Below screenshot shows the partial output of above command:

	![](pic/5-2-1.png)

Here is a log of execution of above command:

~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook verify_fabric.yml --tags "underlay"

PLAY [leaf, jinja2_leaf] *********************************************************************************************************************************************************************************************

TASK [verify underlay] ***********************************************************************************************************************************************************************************************
ok: [198.18.4.101]
ok: [198.18.4.104]
ok: [198.18.4.103]

TASK [debug] *********************************************************************************************************************************************************************************************************
ok: [198.18.4.101] => {
    "underlay_output.stdout_lines": [
        [
            "OSPF Process ID 1 VRF default",
            " Total number of neighbors: 2",
            " Neighbor ID     Pri State            Up Time  Address         Interface",
            " 192.168.0.6       1 FULL/ -          01:27:12 10.0.0.21       Eth1/1 ",
            " 192.168.0.7       1 FULL/ -          01:27:12 10.0.128.5      Eth1/2"
        ],
        [
            "BGP summary information for VRF default, address family IPv4 Unicast",
            "BGP router identifier 192.168.0.8, local AS number 65000",
            "BGP table version is 6, IPv4 Unicast config peers 2, capable peers 2",
            "0 network entries and 0 paths using 0 bytes of memory",
            "BGP attribute entries [0/0], BGP AS path entries [0/0]",
            "BGP community entries [0/0], BGP clusterlist entries [4/16]",
            "",
            "Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd",
            "192.168.0.6     4 65000     161     106        6    0    0 01:23:17 0         ",
            "192.168.0.7     4 65000     161     106        6    0    0 01:23:15 0"
        ],
        [
            "PIM Neighbor Status for VRF \"default\"",
            "Neighbor        Interface            Uptime    Expires   DR       Bidir-  BFD    ECMP Redirect",
            "                                                         Priority Capable State     Capable",
            "10.0.0.21       Ethernet1/1          01:23:08  00:01:35  1        yes     n/a     no",
            "10.0.128.5      Ethernet1/2          01:23:07  00:01:31  1        yes     n/a     no"
        ]
    ]
}
ok: [198.18.4.104] => {
    "underlay_output.stdout_lines": [
        [
            "OSPF Process ID 1 VRF default",
            " Total number of neighbors: 2",
            " Neighbor ID     Pri State            Up Time  Address         Interface",
            " 192.168.0.6       1 FULL/ -          1d04h    10.0.128.1      Eth1/1 ",
            " 192.168.0.7       1 FULL/ -          1d04h    10.0.128.17     Eth1/2"
        ],
        [
            "BGP summary information for VRF default, address family IPv4 Unicast",
            "BGP router identifier 192.168.0.11, local AS number 65000",
            "BGP table version is 5, IPv4 Unicast config peers 2, capable peers 2",
            "0 network entries and 0 paths using 0 bytes of memory",
            "BGP attribute entries [0/0], BGP AS path entries [0/0]",
            "BGP community entries [0/0], BGP clusterlist entries [4/16]",
            "",
            "Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd",
            "192.168.0.6     4 65000     672     662        5    0    0 07:03:12 0         ",
            "192.168.0.7     4 65000    1441    1433        5    0    0 22:36:01 0"
        ],
        [
            "PIM Neighbor Status for VRF \"default\"",
            "Neighbor        Interface            Uptime    Expires   DR       Bidir-  BFD    ECMP Redirect",
            "                                                         Priority Capable State     Capable",
            "10.0.128.1      Ethernet1/1          08:51:43  00:01:28  1        yes     n/a     no",
            "10.0.128.17     Ethernet1/2          22:36:23  00:01:23  1        yes     n/a     no"
        ]
    ]
}
ok: [198.18.4.103] => {
    "underlay_output.stdout_lines": [
        [
            "OSPF Process ID 1 VRF default",
            " Total number of neighbors: 2",
            " Neighbor ID     Pri State            Up Time  Address         Interface",
            " 192.168.0.6       1 FULL/ -          01:27:11 10.0.0.29       Eth1/1 ",
            " 192.168.0.7       1 FULL/ -          01:27:10 10.0.128.13     Eth1/2"
        ],
        [
            "BGP summary information for VRF default, address family IPv4 Unicast",
            "BGP router identifier 192.168.0.10, local AS number 65000",
            "BGP table version is 6, IPv4 Unicast config peers 2, capable peers 2",
            "0 network entries and 0 paths using 0 bytes of memory",
            "BGP attribute entries [0/0], BGP AS path entries [0/0]",
            "BGP community entries [0/0], BGP clusterlist entries [4/16]",
            "",
            "Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd",
            "192.168.0.6     4 65000     148     107        6    0    0 01:23:17 0         ",
            "192.168.0.7     4 65000     150     107        6    0    0 01:23:17 0"
        ],
        [
            "PIM Neighbor Status for VRF \"default\"",
            "Neighbor        Interface            Uptime    Expires   DR       Bidir-  BFD    ECMP Redirect",
            "                                                         Priority Capable State     Capable",
            "10.0.0.29       Ethernet1/1          01:23:09  00:01:37  1        yes     n/a     no",
            "10.0.128.13     Ethernet1/2          01:23:08  00:01:23  1        yes     n/a     no"
        ]
    ]
}

PLAY RECAP ***********************************************************************************************************************************************************************************************************
198.18.4.101               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
198.18.4.103               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
198.18.4.104               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

~~~

Next:

- Run **verify_fabric.yml** playbook and verify the output for **overlay** using the respective tag in the command (as shown below):

```
[root@rhel7-tools EVPN-Ansible]# ansible-playbook verify_fabric.yml --tags "overlay"
```

***
<font style=background-color:yellow>The output shows nve tunnel peer, host route in bgp EVPN from all leaf switches</font>
***

* Below screenshot of the partial output of above command:

	![](pic/5-2-2.png)

- Below shows the complete log output of execution of above playbook command.   Verify the output for **vne vni** status, **vne** dynamic neighbors, type host **mac+ip evpn route update for each L2VNI, l2fib** information.

~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook verify_fabric.yml --tags "overlay"

PLAY [leaf, jinja2_leaf] *********************************************************************************************************************************************************************************************

TASK [Verify Overlay] ************************************************************************************************************************************************************************************************
ok: [198.18.4.104]
ok: [198.18.4.103]
ok: [198.18.4.101]

TASK [debug] *********************************************************************************************************************************************************************************************************
ok: [198.18.4.104] => {
    "overlay_output.stdout_lines": [
        [
            "Codes: CP - Control Plane        DP - Data Plane          ",
            "       UC - Unconfigured         SA - Suppress ARP        ",
            "       SU - Suppress Unknown Unicast",
            " ",
            "Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags",
            "--------- -------- ----------------- ----- ---- ------------------ -----",
            "nve1      50140    239.0.0.140       Up    CP   L2 [140]                  ",
            "nve1      50141    239.0.0.141       Up    CP   L2 [141]                  ",
            "nve1      50999    n/a               Up    CP   L3 [Tenant-1]"
        ],
        [
            "Interface Peer-IP          State LearnType Uptime   Router-Mac       ",
            "--------- ---------------  ----- --------- -------- -----------------",
            "nve1      192.168.0.18     Up    CP        01:07:55 000c.2997.621c   ",
            "nve1      192.168.0.110    Up    CP        01:23:17 000c.2939.f53f"
        ],
        [
            "IP Route Table for VRF \"Tenant-1\"",
            "'*' denotes best ucast next-hop",
            "'**' denotes best mcast next-hop",
            "'[x/y]' denotes [preference/metric]",
            "'%<string>' in via output denotes VRF <string>",
            "",
            "172.21.140.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 22:39:01, direct",
            "172.21.140.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 22:39:01, local",
            "172.21.140.10/32, ubest/mbest: 1/0",
            "    *via 192.168.0.18%default, [200/0], 01:07:52, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a80012 encap: VXLAN",
            " ",
            "172.21.140.11/32, ubest/mbest: 1/0",
            "    *via 192.168.0.110%default, [200/0], 01:23:18, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a8006e encap: VXLAN",
            " ",
            "172.21.141.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 22:38:59, direct",
            "172.21.141.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 22:38:59, local",
            "172.21.141.11/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.11, Vlan141, [190/0], 07:04:08, hmm"
        ],
        [
            "BGP routing table information for VRF default, address family L2VPN EVPN",
            "BGP table version is 458, Local Router ID is 192.168.0.11",
            "Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best",
            "Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected",
            "Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup",
            "",
            "   Network            Next Hop            Metric     LocPrf     Weight Path",
            "Route Distinguisher: 192.168.0.8:32907",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[0]:[0.0.0.0]/216",
            "                      192.168.0.18                      100          0 i",
            "* i                   192.168.0.18                      100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i",
            "* i                   192.168.0.18                      100          0 i",
            "",
            "Route Distinguisher: 192.168.0.10:32907",
            "* i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[0]:[0.0.0.0]/216",
            "                      192.168.0.110                     100          0 i",
            "*>i                   192.168.0.110                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i",
            "* i                   192.168.0.110                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.11:32907    (L2VNI 50140)",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[0]:[0.0.0.0]/216",
            "                      192.168.0.18                      100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[0]:[0.0.0.0]/216",
            "                      192.168.0.110                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.11:32908    (L2VNI 50141)",
            "*>l[2]:[0]:[0]:[48]:[000c.2979.f00d]:[0]:[0.0.0.0]/216",
            "                      192.168.0.111                     100      32768 i",
            "*>l[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100      32768 i",
            "",
            "Route Distinguisher: 192.168.0.11:3    (L3VNI 50999)",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i"
        ],
        [
            "Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link ",
            "(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear",
            "(Ps):Peer Sync (Ro):Re-Originated ",
            "Topology    Mac Address    Prod   Flags         Seq No     Host IP         Next-Hops      ",
            "----------- -------------- ------ ---------- --------------- ---------------",
            "140         0050.56a0.7630 BGP    --            0          172.21.140.10  192.168.0.18   ",
            "140         0050.56a0.b5d1 BGP    --            0          172.21.140.11  192.168.0.110  ",
            "141         000c.2979.f00d HMM    --            0          172.21.141.11  Local"
        ]
    ]
}
ok: [198.18.4.103] => {
    "overlay_output.stdout_lines": [
        [
            "Codes: CP - Control Plane        DP - Data Plane          ",
            "       UC - Unconfigured         SA - Suppress ARP        ",
            "       SU - Suppress Unknown Unicast",
            " ",
            "Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags",
            "--------- -------- ----------------- ----- ---- ------------------ -----",
            "nve1      50140    239.0.0.140       Up    CP   L2 [140]                  ",
            "nve1      50141    239.0.0.141       Up    CP   L2 [141]                  ",
            "nve1      50999    n/a               Up    CP   L3 [Tenant-1]"
        ],
        [
            "Interface Peer-IP          State LearnType Uptime   Router-Mac       ",
            "--------- ---------------  ----- --------- -------- -----------------",
            "nve1      192.168.0.18     Up    CP        01:07:55 000c.2997.621c   ",
            "nve1      192.168.0.111    Up    CP        01:23:20 000c.2951.176f"
        ],
        [
            "IP Route Table for VRF \"Tenant-1\"",
            "'*' denotes best ucast next-hop",
            "'**' denotes best mcast next-hop",
            "'[x/y]' denotes [preference/metric]",
            "'%<string>' in via output denotes VRF <string>",
            "",
            "172.21.140.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 01:24:28, direct",
            "172.21.140.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 01:24:28, local",
            "172.21.140.10/32, ubest/mbest: 1/0",
            "    *via 192.168.0.18%default, [200/0], 01:07:53, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a80012 encap: VXLAN",
            " ",
            "172.21.140.11/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.11, Vlan140, [190/0], 01:23:18, hmm",
            "172.21.141.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 01:24:25, direct",
            "172.21.141.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 01:24:25, local",
            "172.21.141.11/32, ubest/mbest: 1/0",
            "    *via 192.168.0.111%default, [200/0], 01:23:20, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a8006f encap: VXLAN"
        ],
        [
            "BGP routing table information for VRF default, address family L2VPN EVPN",
            "BGP table version is 195, Local Router ID is 192.168.0.10",
            "Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best",
            "Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected",
            "Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup",
            "",
            "   Network            Next Hop            Metric     LocPrf     Weight Path",
            "Route Distinguisher: 192.168.0.8:32907",
            "* i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[0]:[0.0.0.0]/216",
            "                      192.168.0.18                      100          0 i",
            "*>i                   192.168.0.18                      100          0 i",
            "* i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i",
            "*>i                   192.168.0.18                      100          0 i",
            "",
            "Route Distinguisher: 192.168.0.10:32907    (L2VNI 50140)",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[0]:[0.0.0.0]/216",
            "                      192.168.0.18                      100          0 i",
            "*>l[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[0]:[0.0.0.0]/216",
            "                      192.168.0.110                     100      32768 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i",
            "*>l[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100      32768 i",
            "",
            "Route Distinguisher: 192.168.0.10:32908    (L2VNI 50141)",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[0]:[0.0.0.0]/216",
            "                      192.168.0.111                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.11:32908",
            "* i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[0]:[0.0.0.0]/216",
            "                      192.168.0.111                     100          0 i",
            "*>i                   192.168.0.111                     100          0 i",
            "* i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "*>i                   192.168.0.111                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.10:3    (L3VNI 50999)",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100          0 i"
        ],
        [
            "Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link ",
            "(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear",
            "(Ps):Peer Sync (Ro):Re-Originated ",
            "Topology    Mac Address    Prod   Flags         Seq No     Host IP         Next-Hops      ",
            "----------- -------------- ------ ---------- --------------- ---------------",
            "140         0050.56a0.7630 BGP    --            0          172.21.140.10  192.168.0.18   ",
            "140         0050.56a0.b5d1 HMM    --            0          172.21.140.11  Local          ",
            "141         000c.2979.f00d BGP    --            0          172.21.141.11  192.168.0.111"
        ]
    ]
}
ok: [198.18.4.101] => {
    "overlay_output.stdout_lines": [
        [
            "Codes: CP - Control Plane        DP - Data Plane          ",
            "       UC - Unconfigured         SA - Suppress ARP        ",
            "       SU - Suppress Unknown Unicast",
            " ",
            "Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags",
            "--------- -------- ----------------- ----- ---- ------------------ -----",
            "nve1      50140    239.0.0.140       Up    CP   L2 [140]                  ",
            "nve1      50141    239.0.0.141       Up    CP   L2 [141]                  ",
            "nve1      50999    n/a               Up    CP   L3 [Tenant-1]"
        ],
        [
            "Interface Peer-IP          State LearnType Uptime   Router-Mac       ",
            "--------- ---------------  ----- --------- -------- -----------------",
            "nve1      192.168.0.110    Up    CP        01:07:59 000c.2939.f53f   ",
            "nve1      192.168.0.111    Up    CP        01:07:59 000c.2951.176f"
        ],
        [
            "IP Route Table for VRF \"Tenant-1\"",
            "'*' denotes best ucast next-hop",
            "'**' denotes best mcast next-hop",
            "'[x/y]' denotes [preference/metric]",
            "'%<string>' in via output denotes VRF <string>",
            "",
            "172.21.140.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 01:24:29, direct",
            "172.21.140.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.1, Vlan140, [0/0], 01:24:29, local",
            "172.21.140.10/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.140.10, Vlan140, [190/0], 01:24:18, hmm",
            "172.21.140.11/32, ubest/mbest: 1/0",
            "    *via 192.168.0.110%default, [200/0], 01:07:51, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a8006e encap: VXLAN",
            " ",
            "172.21.141.0/24, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 01:24:25, direct",
            "172.21.141.1/32, ubest/mbest: 1/0, attached",
            "    *via 172.21.141.1, Vlan141, [0/0], 01:24:25, local",
            "172.21.141.11/32, ubest/mbest: 1/0",
            "    *via 192.168.0.111%default, [200/0], 01:07:51, bgp-65000, internal, tag 65000 (evpn) segid: 50999 tunnelid: 0xc0a8006f encap: VXLAN"
        ],
        [
            "BGP routing table information for VRF default, address family L2VPN EVPN",
            "BGP table version is 210, Local Router ID is 192.168.0.8",
            "Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best",
            "Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected",
            "Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup",
            "",
            "   Network            Next Hop            Metric     LocPrf     Weight Path",
            "Route Distinguisher: 192.168.0.8:32907    (L2VNI 50140)",
            "*>l[2]:[0]:[0]:[48]:[0050.56a0.7630]:[0]:[0.0.0.0]/216",
            "                      192.168.0.18                      100      32768 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[0]:[0.0.0.0]/216",
            "                      192.168.0.110                     100          0 i",
            "*>l[2]:[0]:[0]:[48]:[0050.56a0.7630]:[32]:[172.21.140.10]/272",
            "                      192.168.0.18                      100      32768 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.8:32908    (L2VNI 50141)",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[0]:[0.0.0.0]/216",
            "                      192.168.0.111                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.10:32907",
            "* i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[0]:[0.0.0.0]/216",
            "                      192.168.0.110                     100          0 i",
            "*>i                   192.168.0.110                     100          0 i",
            "* i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i",
            "*>i                   192.168.0.110                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.11:32908",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[0]:[0.0.0.0]/216",
            "                      192.168.0.111                     100          0 i",
            "* i                   192.168.0.111                     100          0 i",
            "* i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "*>i                   192.168.0.111                     100          0 i",
            "",
            "Route Distinguisher: 192.168.0.8:3    (L3VNI 50999)",
            "*>i[2]:[0]:[0]:[48]:[000c.2979.f00d]:[32]:[172.21.141.11]/272",
            "                      192.168.0.111                     100          0 i",
            "*>i[2]:[0]:[0]:[48]:[0050.56a0.b5d1]:[32]:[172.21.140.11]/272",
            "                      192.168.0.110                     100          0 i"
        ],
        [
            "Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link ",
            "(Dup):Duplicate (Spl):Split (Rcv):Recv(D):Del Pending (S):Stale (C):Clear",
            "(Ps):Peer Sync (Ro):Re-Originated ",
            "Topology    Mac Address    Prod   Flags         Seq No     Host IP         Next-Hops      ",
            "----------- -------------- ------ ---------- --------------- ---------------",
            "140         0050.56a0.7630 HMM    --            0          172.21.140.10  Local          ",
            "140         0050.56a0.b5d1 BGP    --            0          172.21.140.11  192.168.0.110  ",
            "141         000c.2979.f00d BGP    --            0          172.21.141.11  192.168.0.111"
        ]
    ]
}

PLAY RECAP ***********************************************************************************************************************************************************************************************************
198.18.4.101               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
198.18.4.103               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
198.18.4.104               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[root@rhel7-tools EVPN-Ansible]#
~~~

---
### Step 3: Baseline configuration comparison

In this section we will compare the running configuration with baseline configuration for configuration compliance check. The configuration file that we backed in tak 1 will be used as baseline configuration.

In this playbook, you will use **“lookup”** module to find the backup filename generated in Step 1. Then you will use **diff_against** function in **nxos_config** module to compare running configuration.

* On Atom, Open up the project folder `EVPN-Ansible` and create new file under **“EVPN-Ansible”**. Name the new file `verify_config.yml` and enter below data in this playbook:

~~~YAML
---
   - hosts: jinja2_leaf,leaf,jinja2_spine,spine
     vars:
      filename: "{{ lookup('pipe', 'ls backup/{{ inventory_hostname}}_config.*')}}"
     tasks:
       - name: configure compliance
         register: diff_config
         cisco.nxos.nxos_config:
           diff_against: intended
           intended_config: "{{ lookup('file', '{{filename}}') }}"
~~~

* **Click** `File` and `Save` . This will save the playbook, and also ftp the playbook to Ansible server using pre-configured “**remote-sync**” package.

* **Before** you run this playbook, **SSH** into **leaf-4** to make some configuration changes by issuing below commands:
~~~
config t
no router bgp 65000
copy run start
end
~~~

Below screenshot shows the output of above command:

![](pic/5-3-0.png)


* On the Ansible server (via MTputty SSH session), run the playbook for configuration compliance check by executing `ansible-playbook --diff verify_config.yml` as shown below below:

```
[root@rhel7-tools EVPN-Ansible]# ansible-playbook --diff verify_config.yml
```

***
<font style=background-color:yellow>The delta between current running config and base line config are highlighted in <font style=color:limegreen>RED</font> from the result</font>
***

* Below partial screenshot shows the output of above command:

	![](pic/5-3-1.png)

* Bring leaf-4 back to the baseline config by executing `ansible-playbook jinja2_fabric.yml --limit=198.18.4.104` command as shown below:

~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook jinja2_fabric.yml --limit=198.18.4.104
~~~

* Below screenshot shows the output of above command.  You can also log into leaf-4 and verify that bgp configurations are back:

	![](pic/5-3-2.png)

---
### Step 4: Add new VNI

In this section, we will introduce following new VNI into the VXLAN fabric.

|VLAN ID|	VLAN Name	|	VNI|	IP_Add|	mask|	Mcast|
|---|---|---|---|---|---|
|200|	L2-VNI-200-Tenant1|	50200|	172.21.200.1|	24|	239.0.0.200|
201	|L2-VNI-201-Tenant1|	50201|	172.21.201.1|	24|	239.0.0.201|

- First we will creat a new role, and name it **“vni_provision”** under folder roles using **ansible-galaxy** using below commands on the Ansible node (using MTputty via SSH connection):

~~~
[root@rhel7-tools EVPN-Ansible]# cd /root/EVPN-Ansible/roles/
[root@rhel7-tools roles]# ansible-galaxy init vni_provision
~~~

- Verify vni_provision was created successfully

- Ansible-galaxy init will create new role with base role structure and empty **main.yml** file as role requires.  


- Switch to **“Atom”** and sync the new created folders between Ansible node and remote desktop. Right click on project folder **“EVPN-Ansible”**, open **“Remote Sync”** select **“Download Folder”**

	![](pic/5-4-1.png)

- Edit variable file **main.yml** for **“vni_provision”** role under “**/root/EVPN-Ansible/roles/vni_provision/vars**” and enter below data.  Make sure to click `File` and `Save` on Atom to push this to Ansible server:

```YAML
---
# vars file for vni_provision
  L2VNI:
  - { vlan_id: 200, vni: 50200, ip_add: 172.21.200.1, mask: 24, vlan_name: L2-VNI-200-Tenant1, mcast: 239.0.0.200 }
  - { vlan_id: 201, vni: 50201, ip_add: 172.21.201.1, mask: 24, vlan_name: L2-VNI-201-Tenant1, mcast: 239.0.0.201 }
```

![](pic/5-4-2.png)

- Edit playbook file **main.yml** for **“vni_provision”** role under **“/root/EVPN-Ansible/roles/vni_provision/tasks”** and enter below data.   Make sure to click `File` and `Save` on Atom to push this to Ansible server:


```YAML
---
# tasks file for vni_provision
       - name: Configure VLAN to VNI
         cisco.nxos.nxos_vlan:
           vlan_id: "{{ item.vlan_id }}"
           mapped_vni: "{{ item.vni }}"
           name: "{{ item.vlan_name }}"
         with_items:
         - "{{ L2VNI }}"
       - name: Configure L2VNI
         cisco.nxos.nxos_interface:
           interface: vlan"{{ item.vlan_id }}"
         with_items: "{{ L2VNI }}"
       - name: Assign interface to Tenant VRF
         cisco.nxos.nxos_vrf_interface:
           vrf: Tenant-1
           interface: "vlan{{ item.vlan_id }}"
         with_items:
         - "{{ L2VNI }}"
       - name: Configure SVI IP
         cisco.nxos.nxos_l3_interfaces:
           config:
             - name: "vlan{{ item.vlan_id }}"
               ipv4:
                 - address: "{{ item.ip_add }}/{{ item.mask }}"
         with_items: "{{ L2VNI }}"
       - name: Configure L2VNI SVI
         cisco.nxos.nxos_interface:
           interface: vlan"{{ item.vlan_id }}"
           fabric_forwarding_anycast_gateway: true
         with_items: "{{ L2VNI }}"
       - name: Configure L2VNI to VTEP
         cisco.nxos.nxos_vxlan_vtep_vni:
            interface: nve1
            vni: "{{ item.vni }}"
            multicast_group: "{{ item.mcast }}"
         with_items: "{{ L2VNI }}"
       - name: Configure L2VNI RD/RT
         cisco.nxos.nxos_evpn_vni:
          vni: "{{ item.vni }}"
          route_distinguisher: auto
          route_target_both: auto
         with_items: "{{ L2VNI }}"

```

this is shown in below screenshot:

![](pic/5-4-3.png)

- Switch to “Atom”  create new playbook **‘vni_provision.yml’** under project folder **EVPN-Ansible** and enter below data.  Make sure to click `File` and `Save` on Atom to push this to Ansible server:

```YAML
---
- hosts: leaf,jinja2_leaf
  connection: local
  roles:
    - vni_provision
```

- Run playbook **vni_provision.yml** to add new VNIs on the fabric by issuing `ansible-playbook vni_provision.yml` command as shown below:

~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook vni_provision.yml
~~~

Below screenshot shows the output of above command:

![](pic/5-4-4.png)

- Switch to **MTPutty** and connect to **leaf-4** (SSH connection0, verify the change on leaf switches by issuing below command:
```
show nve vni
```

Notice the new created L2VNI as shown in below screenshot:

![](pic/5-4-5-new.png)



#Congratulation! You have completed VXLAN Fabric Lab.
