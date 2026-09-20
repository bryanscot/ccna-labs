# Lab 01 — VLAN & Inter-VLAN Routing (Router-on-a-Stick)

> **Topics:** VLANs · 802.1Q trunking · Subinterfaces · Inter-VLAN routing  
> **Difficulty:** Beginner  
> **Tools:** Packet Tracer  
> **Exam domain:** CCNA 200-301 — 2.0 Network Access, 3.0 IP Connectivity



## 1. Lab Overview

**What it teaches:** How to segment a single physical LAN into multiple broadcast domains (VLANs) at Layer 2, then use one router interface — carved into subinterfaces — to route traffic _between_ those VLANs. This is the classic "router-on-a-stick" design: one physical link, multiple logical ones.

**Exam domain:** CCNA 200-301 Domain 2.0 (Network Access) — VLAN configuration, trunking (802.1Q), and the Domain 3.0 (IP Connectivity) overlap where subinterfaces get IP addresses and route between VLANs.

**Why it matters:** Almost every office network you'll touch as a junior engineer is VLAN-segmented — separating staff, guest Wi-Fi, and management traffic onto different broadcast domains for both performance and security. Router-on-a-stick is usually the first inter-VLAN routing design engineers learn before moving to Layer 3 switches.

**Difficulty:** Beginner.

---

## 2. Topology

**Devices needed:**

- 1x router (R1) — needs a Fast/GigabitEthernet interface that supports subinterfaces
- 1x Layer 2 switch (SW1)
- 3x end devices (PC1, PC2, PC3) — one per VLAN

**Physical connections:**

- R1 Gi0/0 → SW1 Gi0/1 (this link becomes an 802.1Q trunk)
- SW1 Fa0/1 → PC1 (VLAN 10 — Sales)
- SW1 Fa0/2 → PC2 (VLAN 20 — IT)
- SW1 Fa0/3 → PC3 (VLAN 30 — Guest)

**ASCII diagram:**

```
                 Gi0/0                 Gi0/1
        R1  ───────────────────  SW1  ───────┬───────┬───────┐
   (subinterfaces                          Fa0/1    Fa0/2   Fa0/3
    .10 .20 .30)                             │        │        │
                                            PC1      PC2      PC3
                                          VLAN 10   VLAN 20  VLAN 30
                                           Sales       IT      Guest
```

**IP addressing scheme:**

|VLAN|Name|Subnet|Gateway (on R1 subinterface)|Host example|
|---|---|---|---|---|
|10|Sales|192.168.10.0 /24|192.168.10.1|PC1: .10|
|20|IT|192.168.20.0 /24|192.168.20.1|PC2: .10|
|30|Guest|192.168.30.0 /24|192.168.30.1|PC3: .10|

Native VLAN stays as VLAN 1 (default) and carries no host traffic — this is deliberate, see the gotcha note in Step 6.

---

## 3. Step-by-Step Configuration

**Step 1 — On SW1: create the VLANs**

```
SW1(config)# vlan 10
SW1(config-vlan)# name SALES
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name IT
SW1(config-vlan)# exit
SW1(config)# vlan 30
SW1(config-vlan)# name GUEST
SW1(config-vlan)# exit
```

_Why:_ A VLAN has to exist in the switch's VLAN database before you can assign a port to it. Naming them isn't functionally required, but it's what makes `show vlan brief` readable six months from now.

**Step 2 — On SW1: assign access ports**

```
SW1(config)# interface fa0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

SW1(config)# interface fa0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# exit

SW1(config)# interface fa0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# exit
```

_Why:_ `switchport mode access` explicitly locks the port as an access port (untagged, single VLAN) rather than letting DTP (Dynamic Trunking Protocol) negotiate it. Being explicit here avoids a whole category of "why is this port acting weird" problems later.

**Step 3 — On SW1: configure the trunk to R1**

```
SW1(config)# interface gi0/1
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20,30
SW1(config-if)# exit
```

_Why:_ `mode trunk` makes the port carry tagged traffic for multiple VLANs. `trunk allowed vlan` restricts it to only the VLANs you actually want crossing that link — a security and hygiene practice, not just a formality. (Some switch platforms don't need the `encapsulation dot1q` line because 802.1Q is the only option; Packet Tracer's 2960 image still asks for it.)

**Step 4 — On R1: enable the physical interface**

```
R1(config)# interface gi0/0
R1(config-if)# no shutdown
R1(config-if)# exit
```

_Why:_ The physical interface itself needs no IP address — it's just the carrier. All addressing happens on the subinterfaces below. Forgetting `no shutdown` here is the single most common reason this lab "doesn't work" on the first try.

**Step 5 — On R1: create and address the subinterfaces**

```
R1(config)# interface gi0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit

R1(config)# interface gi0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit

R1(config)# interface gi0/0.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit
```

_Why:_ `encapsulation dot1Q 10` tells this subinterface "I only handle frames tagged with VLAN 10." The subinterface number (`.10`) is cosmetic — it's the `encapsulation` line that actually binds it to a VLAN tag. A very common beginner mistake is assuming the subinterface number does the binding; it doesn't.

**Step 6 — On the PCs: set IP addresses**

```
PC1: 192.168.10.10 /24, gateway 192.168.10.1
PC2: 192.168.20.10 /24, gateway 192.168.20.1
PC3: 192.168.30.10 /24, gateway 192.168.30.1
```

_Gotcha:_ If you're tempted to put a PC on native VLAN 1 to "save a subinterface," don't — untagged native-VLAN traffic on a trunk is a known VLAN-hopping attack vector. Keeping VLAN 1 empty of user traffic is a best-practice habit worth building now, even in a lab.

---

## 4. Verification

**On SW1:**

```
SW1# show vlan brief
```

_Expect:_ VLANs 10, 20, 30 listed with the correct ports (Fa0/1, Fa0/2, Fa0/3) under each. If a port shows up under VLAN 1 instead of where you expect, you skipped or mistyped Step 2.

```
SW1# show interfaces trunk
```

_Expect:_ Gi0/1 listed as trunking, with VLANs 10,20,30 in both the "allowed" and "active" columns. **What this tells you:** if a VLAN is in "allowed" but missing from the "active" list, that VLAN doesn't exist in the switch's VLAN database yet — go back to Step 1.

**On R1:**

```
R1# show ip interface brief
```

_Expect:_ Gi0/0, Gi0/0.10, Gi0/0.20, and Gi0/0.30 all showing `up/up`. If the physical Gi0/0 shows `administratively down`, you missed `no shutdown` in Step 4 — and every subinterface riding on it will be down too, even if individually configured correctly.

```
R1# show ip route connected
```

_Expect:_ Three connected routes, one per /24 subnet. **What this tells you:** this is the router's proof that it knows how to reach each VLAN directly — if inter-VLAN ping fails later, this is the first place to confirm the router even has the destination subnet in its table.

**From the PCs:**

```
PC1> ping 192.168.20.10
```

_Expect:_ Success. This ping leaves PC1, hits SW1 tagged as VLAN 10, crosses the trunk to R1, gets routed from the .10 subinterface to the .20 subinterface, crosses back over the trunk tagged as VLAN 20, and arrives at PC2. If this fails but pinging the router's own gateway IPs succeeds, the problem is almost always a missing VLAN in `trunk allowed vlan` on Step 3.

---

## 5. Break It on Purpose (Critical Section)

**Break #1 — Remove a VLAN from the trunk's allowed list**

- **What to change:**
    
    ```
    SW1(config)# interface gi0/1
    SW1(config-if)# switchport trunk allowed vlan remove 20
    ```
    
- **Symptom:** PC2 (VLAN 20) can no longer reach anything outside its own VLAN — not even its default gateway — while PC1 and PC3 are unaffected.
- **Diagnose it:** `show interfaces trunk` on SW1 — VLAN 20 will be missing from the active list even though it still exists in `show vlan brief`. This is the tell that it's a trunk-filtering problem, not a missing-VLAN problem.
- **Fix it:**
    
    ```
    SW1(config-if)# switchport trunk allowed vlan add 20
    ```
    

**Break #2 — Wrong encapsulation VLAN on a subinterface**

- **What to change:** On R1, re-tag the IT subinterface incorrectly:
    
    ```
    R1(config)# interface gi0/0.20
    R1(config-subif)# encapsulation dot1Q 21
    ```
    
- **Symptom:** PC2 can't reach its gateway (192.168.20.1) at all — the interface shows `up/up` in `show ip interface brief`, which makes this a sneaky one because it _looks_ healthy.
- **Diagnose it:** `show interfaces gi0/0.20` will show `802.1Q Virtual LAN, VLAN 21` in the header — the mismatch between the tag the switch is sending (VLAN 20) and what the router is listening for (VLAN 21) is invisible in `show ip interface brief`, which is exactly why relying on one command is a trap.
- **Fix it:** Set it back to `encapsulation dot1Q 20`.

**Break #3 — Duplicate/incorrect subnet mask on a subinterface**

- **What to change:**
    
    ```
    R1(config)# interface gi0/0.30
    R1(config-subif)# ip address 192.168.30.1 255.255.255.128
    ```
    
- **Symptom:** PC3, if addressed at .10, still works — but a device added later at, say, .140 can't reach the gateway, and it's not obvious why since "the subnet looks right."
- **Diagnose it:** `show ip interface brief` won't flag this — you have to calculate the actual usable range for /25 (192.168.30.1–192.168.30.126) and realize .140 falls outside it. This is a good one for building the habit of checking the mask, not just the address.
- **Fix it:** Restore `255.255.255.0`.

---

## 6. Interview-Ready Takeaways

- I configured 802.1Q trunking between a switch and a router and used router-on-a-stick with subinterfaces to route between three VLANs.
- I learned that a subinterface's number is cosmetic — it's the `encapsulation dot1Q` command that actually binds it to a VLAN tag.
- I saw firsthand why leaving the native VLAN empty of user traffic is a security best practice, not just a convention.
- I built a troubleshooting habit of checking `show interfaces trunk` before assuming a routing problem, since a lot of "routing" symptoms are actually Layer 2 trunk-filtering issues.
- I practiced diagnosing a fault that looked healthy at a glance (`up/up`) but had a mismatched VLAN tag underneath — a reminder that one show command is rarely the whole picture.

**Likely interview questions:**

1. _"Walk me through what happens to a frame from PC1 to PC2 in a router-on-a-stick setup."_ Model answer: PC1 sends an untagged frame to the switch. The access port on VLAN 10 tags it 802.1Q VLAN 10 and forwards it out the trunk to the router. The router's Gi0/0.10 subinterface strips the tag, routes the packet at Layer 3 to the 192.168.20.0/24 network, then Gi0/0.20 re-tags it VLAN 20 and sends it back over the same physical trunk to the switch, which strips the tag and delivers it untagged to PC2's access port.
    
2. _"Why can't two VLANs communicate without a router or Layer 3 switch?"_ Model answer: VLANs are separate broadcast domains at Layer 2 — a switch by itself has no Layer 3 routing table, so it can't forward traffic between IP subnets. You need something doing Layer 3 forwarding — a router or a switch with routing enabled (SVIs) — to move traffic between VLANs.
    
3. _"What's the downside of router-on-a-stick compared to a Layer 3 switch doing inter-VLAN routing?"_ Model answer: All inter-VLAN traffic is squeezed through a single physical link and the router's forwarding capacity, so it doesn't scale well for high-traffic environments. Layer 3 switches route at wire speed in hardware and don't have that single-link bottleneck, which is why router-on-a-stick is more common in small networks or labs than in production enterprise cores.
    

---

## 7. Wireshark Moment

**What to capture:** Place a capture point on the trunk link between SW1 and R1 (in Packet Tracer, this is the "simple PDU" + inspect, or in GNS3 you can right-click the link and start a Wireshark capture).

**Filter to use:** `vlan` (isolates 802.1Q-tagged frames) or more specifically `vlan.id == 20` to isolate just the IT VLAN's traffic.

**What to look for:** Open a captured frame and expand the "802.1Q Virtual LAN" layer sitting between the Ethernet header and the IP header. You'll see the VLAN ID field carrying the tag — this is the moment the abstract "VLANs tag frames" explanation becomes a real 12-bit field you can point at. It's also the fastest way to _prove_ Break #2 above: capture on the trunk while PC2 pings its gateway and you'll see frames leaving tagged VLAN 20 that the router is silently dropping because it's listening for VLAN 21.

---

## 8. Stretch Goals (Optional)

- **Add DHCP:** Configure R1 as a DHCP server with a separate pool per VLAN (`ip dhcp pool SALES`, etc.), plus `ip helper-address` isn't even needed here since the router _is_ the DHCP server — a good contrast to set up later when you study DHCP relay in a topology where the server is remote.
- **Add a Layer 3 switch alternative:** Rebuild the same topology using a multilayer switch with SVIs (`interface vlan 10` + `ip routing`) instead of router-on-a-stick, and compare `show ip route` output between the two designs — this sets up the "why L3 switches scale better" interview answer with hands-on evidence instead of just theory.

---
---
