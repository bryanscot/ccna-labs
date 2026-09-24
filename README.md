# CCNA Labs

Hands-on Packet Tracer labs built while working through CCNA 200-301 — each 
one configured, verified, broken on purpose, and documented.

The goal isn't just to configure devices. It's to understand *why* each 
command works, *what* breaks when it's wrong, and *how* to diagnose a fault 
from the symptom alone — the way you'd actually encounter it in the field.

---

## Labs

| # | Lab | Topics | Difficulty |
|---|---|---|---|
| 01 | [VLAN & Inter-VLAN Routing (Router-on-a-Stick)](01-vlan-intervlan-router-on-a-stick/) | VLANs, 802.1Q trunking, subinterfaces | Beginner |
| 02 | [Static vs. OSPF Routing Comparison](02-static-vs-ospf/) | Static routing, OSPF, administrative distance | Intermediate |
| 03 | [NAT/PAT for Internet Access Simulation](03-nat-pat/) | NAT, PAT, translation tables | Beginner–Intermediate |
| 04 | [Standard & Extended ACL Security Lab](04-acl-security/) | ACLs, wildcard masks, VTY restrictions | Intermediate |
| 05 | [DHCP Server Configuration on a Router](05-dhcp-server/) | DHCP pools, exclusions, APIPA | Beginner |
| 06 | [Troubleshooting a Broken OSPF Neighbor Adjacency](06-ospf-troubleshooting/) | OSPF adjacency faults, timers, MTU | Advanced |

---

## What Each Lab Includes

Every lab in this repo follows the same structure:

- **Overview** — what the lab teaches, which CCNA exam domain it maps to, and why it matters in the real world
- **Topology** — devices, physical connections, and IP addressing scheme
- **Step-by-step configuration** — the exact CLI commands with an explanation of *why* each one is used
- **Verification** — the `show` commands and ping tests that confirm the config is working, with what the output actually means
- **Break It on Purpose** — deliberate faults introduced and diagnosed from scratch, because troubleshooting is what separates a lab-doer from an engineer
- **Stretch Goals** — optional extensions for depth once the basics work

---

## Tools Used

- **Cisco Packet Tracer** — topology building and IOS configuration
- **Wireshark** — packet-level verification and protocol analysis

Where a lab references GNS3, it's noted in that lab's overview.

---

## About Me

Brian Waweru — Junior Network Engineer based in Nairobi, Kenya.  
Currently working toward CCNA 200-301.

- Portfolio: https://brian-networking-portfolio.vercel.app
- GitHub: [github.com/bryanscot](https://github.com/bryanscot)
- LinkedIn: [linkedin.com/in/brian-waweru](https://www.linkedin.com/in/brian-waweru-a08141264)
