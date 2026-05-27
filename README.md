[README.md](https://github.com/user-attachments/files/28292025/README.md)
# openstack-private-cloud# Private Cloud Infrastructure with OpenStack

Deploying a 3-node OpenStack private cloud on bare-metal Ubuntu Server as part of a Distributed Computing course project at UIT–VNU.

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Controller Node│    │  Compute Node   │    │  Storage Node   │
│                 │    │                 │    │                 │
│  Keystone       │    │  Nova-Compute   │    │  Cinder-Volume  │
│  Nova API       │    │  Neutron Agent  │    │                 │
│  Glance         │    │  KVM / QEMU     │    │  Ubuntu 24.04   │
│  Neutron Server │    │                 │    │  RAM: 2GB       │
│  Cinder API     │    │  Ubuntu 24.04   │    │  HDD: 100GB     │
│  Horizon        │    │  RAM: 4GB       │    └─────────────────┘
│                 │    │  HDD: 100GB     │
│  Ubuntu 24.04   │    └─────────────────┘
│  RAM: 3GB       │
│  HDD: 60GB      │
└─────────────────┘
```

**Network layout**

| Network       | Subnet          | Purpose                        |
|---------------|-----------------|--------------------------------|
| Management    | 10.10.10.0/24   | API communication between nodes|
| Data (VXLAN)  | 10.10.20.0/24   | VM tenant traffic overlay      |
| Provider      | External        | Floating IP / internet access  |

## Services Deployed

| Service   | Component  | Role                              |
|-----------|------------|-----------------------------------|
| Keystone  | Identity   | Authentication & authorization    |
| Nova      | Compute    | VM lifecycle management (KVM)     |
| Glance    | Image      | VM image storage & retrieval      |
| Neutron   | Network    | Virtual networking (VXLAN / ML2)  |
| Cinder    | Block Storage | Persistent volumes for VMs     |
| Horizon   | Dashboard  | Web UI for users & admins         |

## Key Configurations

**Neutron — Self-Service Network (VXLAN)**
```ini
# ml2_conf.ini
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
```

**Nova — KVM Hypervisor**
```ini
# nova.conf
[DEFAULT]
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[libvirt]
virt_type = kvm
```

**Cinder — LVM Block Storage**
```bash
# Create physical volume on storage node
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
```

## What Was Built

- Provisioned a 3-node OpenStack cluster on bare-metal Ubuntu Server 24.04 supporting VM creation and isolated tenant networks
- Configured Neutron networking with VXLAN overlay, enabling isolated tenant networks and floating IP assignment for 10+ test VMs
- Automated node configuration using Ansible playbooks, reducing manual setup time from ~4 hours to ~30 minutes per node
- Deployed Horizon dashboard for web-based VM management, image upload, and network configuration

## Verification

```bash
# List running services
openstack service list

# Check compute nodes
openstack compute service list

# Check network agents
openstack network agent list

# List available images
openstack image list

# Launch a test VM
openstack server create \
  --flavor m1.tiny \
  --image cirros \
  --network demo-net \
  test-vm
```

## Tech Stack

`OpenStack` `Ubuntu Server 24.04` `KVM / QEMU` `Neutron` `Nova` `Cinder` `Ansible` `MariaDB` `RabbitMQ`
