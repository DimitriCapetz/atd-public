# Lab Development Notes

This file is the persistent handoff for the prospective-customer lab work. Update it as decisions are made and changes are implemented.

## Objective

Add several approachable, hands-on Arista Networks labs to this ATD repository for prospective customers. Labs should fit the repository's existing topology, menu, configlet, and lab-guide conventions.

## Repository findings

- `topologies/campus` contains campus-oriented labs and menu definitions.
- `topologies/dual-datacenter` contains leaf-spine, VXLAN/EVPN, CloudVision, and AVD-oriented labs.
- `topologies/routing` contains routing labs including IS-IS, BGP, Segment Routing, and L3VPN.
- `topologies/wan` contains WAN-oriented lab content.
- Customer-facing lab selections are exposed through files under `files/menus/`.
- Device state for selectable labs is assembled from named configlets, generally with a base configlet plus a lab-specific configlet.
- Longer guided exercises are stored as Markdown, including the AVD EVPN/VXLAN guide under the dual-datacenter topology.

## Current lab

- Name: **Core Backbone**
- Hosting topology: `topologies/routing`
- Status: implemented and ready for review

## Initial proposal

Potential introductory labs discussed so far:

1. EOS and network automation fundamentals
2. Layer 3 leaf-spine with BGP and EVPN
3. CloudVision change control and telemetry

These are proposals only; the target topology, audience, and exact technologies still need to be confirmed.

## Core Backbone requirements

- Goal: mimic a customer core-backbone environment that learners configure from a blank or baseline state.
- Topology: five core locations, with two devices per location (10 core nodes total), arranged in a ring.
- Clients: additional routing-topology nodes should be fully configured to simulate endpoints.
- Starting state: retain each device base configuration; enable/disable EOS interfaces as needed to establish the physical topology; leave the core service/routing configuration largely unconfigured.
- Documentation and validation will be supplied elsewhere.
- Repository deliverables: a new routing-topology menu entry and the required configlets.

## Proposed Core Backbone mapping

- Site 1: eos1--eos2; Site 2: eos3--eos4; Site 3: eos8--eos14; Site 4: eos6--eos13; Site 5: eos11--eos12.
- Site-ring links: eos2--eos3, eos4--eos8, eos14--eos6, eos13--eos12, eos11--eos1.
- Prepared client nodes: eos15, eos16, eos17, eos18, and eos20, selected because each has a direct existing link to a core node.
- Unused/support nodes remain available: eos5, eos7, eos9, eos10, and eos19.

## Implementation status

- Added `topologies/routing/files/menus/Core-Backbone.yaml` with a reset lab covering all 20 routing nodes.
- Added `CORE_BACKBONE_EOS1`, `EOS2`, `EOS3`, `EOS4`, `EOS6`, `EOS8`, `EOS11`, `EOS13` configlets; each contains only shutdown commands for interfaces outside the selected ring/client mapping.
- Core nodes use `BaseIPv4_EOSx_BARE`; selected clients use `BaseIPv4_EOSx_BARE` baselines followed by applicable Core Backbone configlets.
- The new menu is automatically discoverable through the routing lab menu selector.
- Validation completed: Core-Backbone YAML parses successfully, includes all 20 nodes, and VLAN 99 configuration checks pass.

## EVPN client requirement

- The Core Backbone lab will emulate a VXLAN EVPN core.
- Client nodes must support testing both Layer 2 extensions (for example, VLANs carried between sites) and Layer 3 extensions (for example, site-specific subnets or routed client handoffs).
- Client configlets should therefore be designed as EVPN service endpoints, not merely point-to-point routed hosts.
- Current selected client placement provides clients at Site 1 (eos17), Site 2 (eos16), and Site 3 (eos15/eos18); Sites 4 and 5 currently have no selected client attachment.

## Confirmed scope

- Not every site requires a client attachment. Keep the current topology and use the clients at Sites 1--3 for VXLAN EVPN Layer 2 and Layer 3 extension exercises.
- Do not redesign the physical topology or add client links for Sites 4 and 5.

## VLAN 100 client implementation

- VLAN 100 is configured on clients eos15, eos16, eos17, eos18, and eos20.
- Each client uses Ethernet1 as a trunk allowing VLAN 100.
- Each client has VRF VLAN100 and an SVI address: eos15 = 10.100.100.15/24, eos16 = .16/24, eos17 = .17/24, eos18 = .18/24, eos20 = .20/24.
- The core menu applies each client BARE baseline followed by its VLAN 100 client configlet.

## VLAN 101 client extension

- Extended the existing eos17 and eos20 client configlets with VLAN 101.
- VLAN 101 uses VRF VLAN101, SVI addresses 10.101.101.17/24 and 10.101.101.20/24, and a VRF default route via 10.101.101.1.
- Their Ethernet1 trunks now allow VLANs 100 and 101.
- No separate VLAN 101 configlet files were created.

## Site and client reference

| Site | Core devices | Client connections |
|---|---|---|
| Site 1 | eos1 / eos2 | eos17 Ethernet1 to eos1 Ethernet6 |
| Site 2 | eos3 / eos4 | eos20 Ethernet1 to eos3 Ethernet6; eos16 Ethernet1 to eos4 Ethernet6 |
| Site 3 | eos8 / eos14 | eos15 Ethernet1 to eos8 Ethernet2; eos18 Ethernet1 to eos8 Ethernet5 |
| Site 4 | eos6 / eos13 | No client |
| Site 5 | eos11 / eos12 | No client |

## VLAN 102 client extension

- Extended the existing eos15 and eos16 client configlets with VLAN 102.
- VLAN 102 uses VRF VLAN102, SVI addresses 10.102.102.15/24 and 10.102.102.16/24, and a VRF default route via 10.102.102.1.
- Their Ethernet1 trunks now allow VLANs 100 and 102.

## VLAN 99 BGP client extension

- Site 2 uses eos4 (ASN 65002) and client eos16 (ASN 65016) over 10.99.2.0/30: core .1, client .2.
- Site 3 uses eos8 (ASN 65003) and client eos18 (ASN 65018) over 10.99.3.0/30: core .1, client .2.
- VLAN 99 is carried on the client trunks in VRF VLAN99.
- Client loopbacks 99.99.99.16/32 and 99.99.99.18/32 are advertised through the respective eBGP sessions.

## Device hostname convention

- Core hostnames: eos1/eos2 = SITE1-CORE-1/-2; eos3/eos4 = SITE2-CORE-1/-2; eos8/eos14 = SITE3-CORE-1/-2; eos6/eos13 = SITE4-CORE-1/-2; eos11/eos12 = SITE5-CORE-1/-2.
- Client hostnames: eos17 = SITE1-CLIENT; eos16/eos20 = SITE2-CLIENT-1/-2; eos15/eos18 = SITE3-CLIENT-1/-2.
- Unused/support nodes eos5, eos7, eos9, eos10, and eos19 retain their EOS hostnames.

## Decisions

- The Core Backbone implementation is ready for commit and push.
- The unrelated `.devcontainer/Dockerfile` change was preserved and excluded from the Core Backbone commit.

## Open questions

- Customer-facing lab instructions and validation will be added separately.

## Work log

### 2026-09-02

- Inspected the repository structure and representative topology, menu, and lab-guide files.
- Confirmed the repository is an Arista Test Drive (ATD) public repository.
- Created this persistent handoff note.
- Added the Core Backbone menu and baseline, core, and client configlets.
- Added VLANs 99, 100, 101, and 102 client service configuration as specified.
- Confirmed configuration checks pass.
