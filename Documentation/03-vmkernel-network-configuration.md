# Task 03 — Configure VMkernel Networks & vMotion Networking

## Objective

Configure and verify the VMkernel networks required for the vSphere infrastructure.

The hosts use separate VMkernel interfaces for:

- Management
- Lab infrastructure
- vMotion

The goal is to prepare the network for future vCenter, cluster, and vMotion operations.

## Environment

| Host | Management VMkernel | Lab VMkernel | vMotion VMkernel |
|---|---|---|---|
| ESXi-01 | `vmk0` — 192.168.122.190 | `vmk1` — 192.168.200.11 | `vmk2` — 192.168.210.11 |
| ESXi-02 | `vmk0` — 192.168.122.135 | `vmk1` — 192.168.200.12 | `vmk2` — 192.168.210.12 |
| ESXi-03 | `vmk0` — 192.168.122.116 | `vmk1` — 192.168.200.13 | `vmk2` — 192.168.210.13 |

### vMotion Network

- Port Group: `vMotion Network`
- Virtual Switch: `vSwitch1`
- VLAN: `0`
- Subnet: `192.168.210.0/24`
- TCP/IP Stack: `Default TCP/IP stack`

## Implementation

### Management VMkernel

The default management VMkernel interface `vmk0` was verified on all three ESXi hosts.

The Management service is enabled on `vmk0`.

Management addresses:

- ESXi-01: `192.168.122.190`
- ESXi-02: `192.168.122.135`
- ESXi-03: `192.168.122.116`

### Lab VMkernel

The Lab Network uses `vmk1`.

The Lab VMkernel interfaces are connected to the shared network created in Task 02.

Lab addresses:

- ESXi-01: `192.168.200.11`
- ESXi-02: `192.168.200.12`
- ESXi-03: `192.168.200.13`

### vMotion Port Group

A new port group named `vMotion Network` was created on `vSwitch1` on all three hosts.

The default security settings were kept:

- Promiscuous Mode: Reject
- MAC Address Changes: Reject
- Forged Transmits: Reject

### vMotion VMkernel

A dedicated VMkernel interface `vmk2` was created on each ESXi host.

The vMotion service was enabled on `vmk2`.

The final configuration is:

- ESXi-01: `192.168.210.11`
- ESXi-02: `192.168.210.12`
- ESXi-03: `192.168.210.13`

## Verification

Connectivity was tested using `vmkping` through the vMotion VMkernel interface.

Example:

`vmkping -I vmk2 192.168.210.12`

All six connectivity tests between the three hosts were successful.

| Source | Destination | Result |
|---|---|---|
| ESXi-01 192.168.210.11 | ESXi-02 192.168.210.12 | Successful |
| ESXi-01 192.168.210.11 | ESXi-03 192.168.210.13 | Successful |
| ESXi-02 192.168.210.12 | ESXi-01 192.168.210.11 | Successful |
| ESXi-02 192.168.210.12 | ESXi-03 192.168.210.13 | Successful |
| ESXi-03 192.168.210.13 | ESXi-01 192.168.210.11 | Successful |
| ESXi-03 192.168.210.13 | ESXi-02 192.168.210.12 | Successful |

## Troubleshooting

The ESXi Host Client `Actions → SSH Console` option opened the local Secure Shell application and attempted to connect to the public IP on TCP/22.

The public IP belongs to the outer Ubuntu VM, so this did not provide direct access to the nested ESXi host.

A dedicated SSH forwarding port was configured for lab administration:

`Public IP:2222 → Ubuntu outer VM → Nested ESXi TCP/22`

This allowed direct access to the ESXi Shell without using noVNC for every command.

The iptables configuration was saved using `iptables-save`, and `netfilter-persistent` was enabled on the three Ubuntu outer hosts to make the SSH forwarding persistent across reboot.

This SSH forwarding is only a lab management method and is not part of the vSphere production network design.

## Important Finding

A successful `vmkping` test proves that the VMkernel interfaces can communicate over the vMotion network.

It does not prove that an actual vMotion operation is working.

A real vMotion test requires:

- vCenter Server
- ESXi hosts managed by vCenter
- A cluster
- Compatible hosts
- Appropriate vSphere licensing
- A running virtual machine
- Required storage and network configuration

The actual live migration test will be performed later after the vCenter and cluster infrastructure are built.

## License

The ESXi hosts use a vSphere 8 Enterprise Plus license that provides the vSphere vMotion feature.

This allowed the vMotion service to be enabled on the VMkernel interfaces.

## Result

The three ESXi hosts now have dedicated VMkernel interfaces for vMotion.

The vMotion network is `192.168.210.0/24`.

All three hosts can communicate with each other through `vmk2`.

The network is ready for the future vCenter cluster and actual vMotion testing.

## What I Learned

- VMkernel interfaces provide dedicated ESXi network functions.
- Management and vMotion traffic can use separate VMkernel interfaces.
- A dedicated vMotion network improves traffic separation.
- `vmkping -I vmk2` can test connectivity through a specific VMkernel interface.
- Network connectivity alone does not prove that vMotion works.
- Actual vMotion requires vCenter and a properly configured cluster.
- SSH access to nested ESXi requires consideration of the NAT architecture.
