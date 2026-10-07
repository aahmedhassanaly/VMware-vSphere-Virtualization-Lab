# Task 02 — Build a Shared ESXi Network Fabric

## Objective

Build shared Layer 2 connectivity between the three nested ESXi hosts.

The original design used separate libvirt networks, so the ESXi hosts could not communicate with each other directly.

## Environment

| Host | Underlay IP | Overlay VMkernel IP |
|---|---|---|
| ESXi-01 | 10.10.20.20 | 192.168.200.11 |
| ESXi-02 | 10.10.20.21 | 192.168.200.12 |
| ESXi-03 | 10.10.20.22 | 192.168.200.13 |

### Network Design

- GCP VPC: Layer 3 underlay
- VXLAN VNI: `100`
- VXLAN UDP port: `4789`
- Overlay subnet: `192.168.200.0/24`
- VXLAN MTU: `1410`
- Underlay MTU: `1460`
- ESXi Lab VMkernel: `vmk1`

## Problem

Each nested ESXi host was connected to a separate libvirt network.

Although the networks used the same IP subnet, they were separate Layer 2 networks.

As a result, the ESXi hosts could not communicate directly with each other.

## Implementation

A VXLAN overlay was built between the three Ubuntu outer hosts.

The network path became:

`Nested ESXi → vnet1 → br-esxi-lab → vxlan100 → ens4 → GCP VPC`

The three ESXi hosts use the Lab Network through `vmk1`.

The overlay addresses are:

- ESXi-01: `192.168.200.11`
- ESXi-02: `192.168.200.12`
- ESXi-03: `192.168.200.13`

Static VXLAN FDB entries were configured for the three underlay peers.

The VXLAN and Linux bridge configuration was also made persistent using a systemd service.

## MTU Consideration

The GCP underlay uses an MTU of `1460`.

VXLAN adds encapsulation overhead, so the VXLAN interface was configured with an MTU of `1410`.

A `vmkping` test using the Lab VMkernel confirmed that the overlay could carry the tested packet size without fragmentation.

## Verification

Connectivity was tested using the ESXi VMkernel interface:

`vmkping -I vmk1`

The tests confirmed connectivity between the ESXi hosts over the overlay network.

### ESXi-01 VMkernel

<img width="1848" height="795" alt="ESXi-01 VMkernel" src="https://github.com/user-attachments/assets/dace23f7-8114-46c8-968b-8173bd3541db" />

### ESXi-02 VMkernel

<img width="1919" height="798" alt="ESXi-02 VMkernel" src="https://github.com/user-attachments/assets/20b50f40-f976-4b8d-b4a4-76d3bbe3cfcc" />

### ESXi-03 VMkernel

<img width="1915" height="855" alt="ESXi-03 VMkernel" src="https://github.com/user-attachments/assets/d3a5e62f-bb77-46fe-a8d1-a375824c3cd8" />

## Persistence Verification

The VXLAN and bridge configuration was verified after reboot.

The following components returned successfully:

- `br-esxi-lab`
- `vxlan100`
- `vnet1`

The persistent libvirt configuration also kept `vnet1` connected to `br-esxi-lab`.

## Important Finding

Using the same IP subnet does not automatically mean that devices are on the same Layer 2 network.

The original networks had the same address range but were isolated because each outer Ubuntu host had its own libvirt network.

VXLAN allowed us to extend the Layer 2 segment across the Layer 3 GCP underlay.

## Result

The three nested ESXi hosts can now communicate through the shared Lab Network:

- ESXi-01: `192.168.200.11`
- ESXi-02: `192.168.200.12`
- ESXi-03: `192.168.200.13`

The network is persistent across reboot and is ready for the next vSphere networking tasks.

## What I Learned

- Layer 2 and Layer 3 are different concepts.
- The same subnet does not always mean the same Layer 2 network.
- VXLAN can extend Layer 2 networks across a Layer 3 underlay.
- VNI identifies a VXLAN segment.
- FDB entries help VXLAN forward Ethernet traffic to remote peers.
- VXLAN adds encapsulation overhead, so MTU must be considered.
- `vmkping -I vmk1` can verify connectivity through a specific ESXi VMkernel interface.
