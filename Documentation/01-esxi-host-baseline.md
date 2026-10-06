# Task 01 — ESXi Host Baseline & Administration

## Objective

Configure and verify the basic settings of the three ESXi hosts before building the vSphere environment.

## Environment

| Host | ESXi Version | Management IP |
|---|---|---|
| ESXi-01 | 8.0.3 Build 24677879 | 192.168.122.190 |
| ESXi-02 | 8.0.3 Build 24677879 | 192.168.122.135 |
| ESXi-03 | 8.0.3 Build 24677879 | 192.168.122.116 |

## Implementation

### NTP Configuration

NTP was configured on the three ESXi hosts using `pool.ntp.org`.

The NTP service was started and verified as running.

### Management Network

The Management VMkernel interface `vmk0` was verified on the hosts.

Example configuration:

- IP Address: `192.168.122.190`
- Subnet Mask: `255.255.255.0`
- Gateway: `192.168.122.1`
- Management: Enabled

### Host Services

Important ESXi services were checked.

The main services were running:

- DCUI
- ntpd
- vpxa
- vmsyslogd

SSH and ESXi Shell remained stopped because they were not required.

## Troubleshooting

During NTP configuration, the Host Client returned:

`Failed - Cannot change the host configuration.`

The NTP server was saved successfully, and the NTP service was started from the ESXi Services page.

The Host Client `Actions` option for starting NTP was not working in this ESXi 8 environment.

## Important Finding

All three hosts use `192.168.122.0/24`, but they are not on the same Layer 2 network.

Each nested ESXi host is connected to a separate `libvirt default` network on its outer Ubuntu VM.

This means the current network design is not ready for:

- vCenter Cluster
- vMotion
- HA
- Shared infrastructure networking

This will be addressed in Task 02.

## Host Health

Hardware sensor data was not available because the nested ESXi hosts do not provide IPMI hardware sensors.

This is expected in a nested virtualization environment.

## Evidence

### NTP Configuration

<img width="1919" height="860" alt="image" src="https://github.com/user-attachments/assets/aed64ec3-8a3a-4aac-b19d-0c3a2fd9691f" />

### Management Network

<img width="1586" height="699" alt="image" src="https://github.com/user-attachments/assets/3f4e202d-d305-4081-a761-fa17154e0a35" />

### ESXi Services
<img width="1919" height="776" alt="image" src="https://github.com/user-attachments/assets/58da73dc-df87-46cd-ad34-f253b2b87d99" />


## Result

The three ESXi hosts have a basic operational baseline.

NTP, Management networking, and important services were verified. The main remaining infrastructure issue is the isolated Layer 2 network design.

## What I Learned

- NTP is important for VMware infrastructure.
- VMkernel interfaces provide ESXi management connectivity.
- The same IP subnet does not always mean the same Layer 2 network.
- Nested ESXi does not provide physical hardware sensor data.

