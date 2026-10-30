# ESXI-IRONCORE

### Standalone VMware Infrastructure Engineering Lab

A hands-on infrastructure engineering project focused on VMware ESXi 8.0, nested virtualization, virtual networking, storage integration, and network resilience — without requiring vCenter Server.

## 1. Project Overview

ESXI-IRONCORE is a practical VMware infrastructure engineering lab designed to demonstrate the deployment, configuration, integration, and validation of a standalone VMware ESXi 8.0 environment.

The project runs ESXi as a nested hypervisor inside VMware Workstation, allowing the implementation of enterprise-inspired virtualization and networking concepts within a controlled laboratory environment.

Unlike traditional VMware environments managed through vCenter Server, this project operates entirely with a single standalone ESXi host using the ESXi Embedded Host Client.

The laboratory focuses on:

- Nested virtualization and hypervisor administration.
- Standard virtual switches and port group configuration.
- Network segmentation and VMkernel networking.
- iSCSI and NFS storage integration.
- Virtual machine deployment and management.
- Network connectivity and link-failure testing.
- Infrastructure troubleshooting and technical documentation.

The objective is to demonstrate practical infrastructure engineering skills through architecture design, implementation, testing, and documentation.

## 2. Project Objectives

The primary objectives are:

1. Deploy and administer VMware ESXi 8.0 without vCenter Server.
2. Build a virtual networking architecture using Standard vSwitches.
3. Separate management, storage, and virtual machine traffic logically.
4. Configure VMkernel interfaces for management and storage connectivity.
5. Integrate external virtual storage using iSCSI and NFS.
6. Deploy and manage Linux virtual machines.
7. Test network connectivity, link failures, and service recovery.
8. Document infrastructure configurations and validation results.

## 3. Infrastructure Architecture

### 3.1 Virtualization Architecture

The laboratory consists of three infrastructure layers:

| Layer             | Component          | Description                                  |
| ----------------- | ------------------ | -------------------------------------------- |
| Physical          | Workstation Host   | Physical computer running VMware Workstation |
| Virtualization    | VMware Workstation | Outer hypervisor                             |
| Nested Hypervisor | VMware ESXi 8.0    | Standalone virtualization platform           |
| Storage           | OpenMediaVault     | Virtual NAS and storage services             |
| Workloads         | Debian Server      | Linux infrastructure testing VM              |

### 3.2 ESXi Host Specifications

| Resource            | Allocation                |
| ------------------- | ------------------------- |
| Hypervisor          | VMware ESXi 8.0           |
| Management Platform | ESXi Embedded Host Client |
| Outer Hypervisor    | VMware Workstation        |
| vCPU                | 4                         |
| RAM                 | 16 GB                     |
| System Disk         | 100 GB                    |
| Network Adapters    | 2 Virtual NICs            |
| Additional Storage  | Separate virtual disk     |
| vCenter Server      | Not deployed              |

### 3.3 Network Topology

The network architecture uses two Standard Virtual Switches connected to two virtual network adapters provided by VMware Workstation.

| Adapter | Virtual Switch | Workstation Network | Purpose                     |
| ------- | -------------- | ------------------- | --------------------------- |
| vmnic0  | vSwitch0       | NAT / Bridged       | Management                  |
| vmnic1  | vSwitch1       | Host-only           | Storage and VM connectivity |

Virtual Switch Design

- `vSwitch0`: Management network.
- `vSwitch1`: Storage and production virtual machine network.

### 3.4 Standard Port Groups

| Port Group    | Virtual Switch | VLAN ID | Purpose              |
| ------------- | -------------- | ------- | -------------------- |
| PG-Management | vSwitch0       | 0       | ESXi management      |
| PG-iSCSI      | vSwitch1       | 0       | Storage connectivity |
| PG-Prod-VMs   | vSwitch1       | 0       | Production test VMs  |

VLAN ID 0 is used for untagged traffic in the initial implementation. VLAN tagging can be introduced later with a compatible VLAN-aware virtual networking environment.

Note: Port Groups on the same Standard Switch using VLAN ID 0 do not provide Layer 2 isolation. Actual VLAN segmentation requires appropriate VLAN tagging and upstream network support.

### 3.5 IP Addressing Plan

The following addressing scheme is proposed for the laboratory:

| Device         | Interface      | IP Address       | Purpose          |
| -------------- | -------------- | ---------------- | ---------------- |
| ESXi Host      | vmk0           | 192.168.10.10/24 | Management       |
| ESXi Host      | vmk1           | 192.168.30.10/24 | Storage          |
| OpenMediaVault | Storage NIC    | 192.168.30.20/24 | Storage services |
| Debian Server  | Production NIC | 192.168.30.30/24 | Linux workload   |

The management subnet must be adapted to the actual VMware Workstation NAT or Bridged network configuration.

## 4. Storage Architecture

Storage integration is implemented through a dedicated OpenMediaVault virtual machine.

### Storage Platform

OpenMediaVault (OMV) provides network storage services to the ESXi host.

| Component          | Configuration  |
| ------------------ | -------------- |
| Storage Server     | OpenMediaVault |
| vCPU               | 2              |
| RAM                | 3 GB           |
| OS Disk            | 20 GB          |
| iSCSI Virtual Disk | 30 GB          |
| NFS Virtual Disk   | 30 GB          |
| Network            | PG-iSCSI       |
| Storage IP         | 192.168.30.20  |

### Storage Protocols

iSCSI

- Configure the Software iSCSI adapter on ESXi.
- Enable iSCSI services on OpenMediaVault.
- Configure dynamic target discovery.
- Rescan storage adapters.
- Discover and format the iSCSI LUN using VMFS 6.

NFS

- Configure an NFS shared directory on OpenMediaVault.
- Configure export permissions.
- Mount the NFS share as an ESXi datastore.
- Validate datastore accessibility.

Important: The OpenMediaVault operating system remains on the local ESXi datastore. Its exported storage is used for learning and testing. This is nested virtual storage, not independent physical SAN storage.

## 5. Virtual Machine Resource Allocation

The initial workload allocation is:

| Virtual Machine       | vCPU | RAM    | Storage     |
| --------------------- | ---- | ------ | ----------- |
| OpenMediaVault        | 2    | 3 GB   | 80 GB total |
| Debian Server         | 2    | 2 GB   | 20 GB       |
| Optional Alpine Linux | 1    | 512 MB | 8 GB        |

These are proposed VM configurations. The optional Alpine workload will be introduced only when sufficient resources are available.

The 16 GB of memory assigned to ESXi is shared between its virtual machines and hypervisor overhead. Actual memory consumption must be monitored.

## 6. Implementation Roadmap

The project is divided into nine implementation phases.

### Phase 01 — ESXi Deployment

- Install VMware ESXi 8.0 inside VMware Workstation.
- Configure nested virtualization.
- Verify CPU, memory, storage, and network resources.
- Access the ESXi Embedded Host Client.

### Phase 02 — Local Storage Configuration

- Attach the additional virtual disk.
- Identify the disk through ESXi Host Client.
- Create a local VMFS datastore.
- Configure the datastore as `DS-LOCAL-01`.

### Phase 03 — Virtual Network Configuration

- Configure `vSwitch0` and `vSwitch1`.
- Assign the appropriate physical uplinks.
- Configure Standard Port Groups.
- Review virtual switch security policies.

### Phase 04 — VMkernel Networking

- Preserve the ESXi management interface.
- Create the storage VMkernel interface.
- Configure static IP addressing.
- Validate connectivity using `vmkping`.

### Phase 05 — OpenMediaVault Deployment

- Deploy OpenMediaVault.
- Configure its virtual network interface.
- Attach dedicated virtual storage disks.
- Configure iSCSI and NFS services.

### Phase 06 — Storage Integration

- Enable Software iSCSI.
- Configure Dynamic Discovery.
- Rescan storage adapters.
- Discover the iSCSI LUN.
- Create a VMFS 6 datastore.
- Mount the NFS datastore.

### Phase 07 — Linux Workload Deployment

- Deploy Debian Server.
- Configure its network interface.
- Validate connectivity.
- Test access to the required infrastructure services.

### Phase 08 — Infrastructure Validation

- Perform network connectivity tests.
- Validate iSCSI discovery.
- Validate NFS datastore mounting.
- Test virtual network disconnection.
- Observe link-state changes and recovery.
- Inspect ESXi logs.

### Phase 09 — Technical Documentation

- Document the architecture.
- Publish network diagrams.
- Capture implementation screenshots.
- Record testing results.
- Document troubleshooting procedures.

## 7. Network and Storage Validation

The following ESXi Shell commands are used during troubleshooting and validation.

### Network Interfaces

List physical network adapters:

```
esxcli network nic list
```

List Standard Virtual Switches:

```
esxcli network vswitch standard list
```

List VMkernel interfaces:

```
esxcli network ip interface list
```

Test storage network connectivity:

```
vmkping -I vmk1 192.168.30.20
```

### Storage

List storage adapters:

```
esxcli storage core adapter list
```

List storage devices:

```
esxcli storage core device list
```

List available datastores:

```
esxcli storage filesystem list
```

List iSCSI adapters:

```
esxcli iscsi adapter list
```

## 8. Network Resilience and Failure Testing

An important part of this project is understanding how the ESXi host and its virtual workloads behave when network connectivity is interrupted.

### Test Scenario

1. Deploy Debian Server on `PG-Prod-VMs`.
2. Start continuous ICMP connectivity tests.
3. Disconnect `vmnic1` through VMware Workstation.
4. Observe the resulting network interruption.
5. Inspect ESXi networking status and system logs.
6. Reconnect the adapter.
7. Validate connectivity recovery.

### Expected Observations

| Test                    | Expected Result                   |
| ----------------------- | --------------------------------- |
| Management connectivity | Operational through vmnic0        |
| Storage connectivity    | Operational through vmnic1        |
| Debian connectivity     | Depends on connected network      |
| vmnic1 disconnection    | External lab network interruption |
| vmnic1 reconnection     | Connectivity recovery             |

Architecture limitation: With one uplink on each Standard Switch, this laboratory does not implement true uplink redundancy. It demonstrates link-failure detection and recovery. Actual uplink failover requires multiple uplinks configured on the same Standard Switch.

VMware HA, DRS, vMotion, and Distributed Switch functionality are outside the scope of this standalone environment.

## 9. Repository Structure

```
ESXI-IRONCORE/
│
├── README.md
│
├── architecture/
│   ├── network-topology.png
│   ├── infrastructure-design.md
│   └── ip-addressing.md
│
├── 01-esxi-deployment/
│   └── installation.md
│
├── 02-virtual-networking/
│   ├── standard-vswitches.md
│   ├── port-groups.md
│   ├── vmkernel-networking.md
│   └── security-policies.md
│
├── 03-storage/
│   ├── openmediavault.md
│   ├── iscsi-configuration.md
│   └── nfs-configuration.md
│
├── 04-virtual-machines/
│   └── debian-server.md
│
├── 05-testing/
│   ├── connectivity.md
│   ├── storage-validation.md
│   └── link-failure-testing.md
│
└── screenshots/
    ├── esxi/
    ├── networking/
    ├── storage/
    └── testing/
```

## 10. Security Considerations

Nested virtualization introduces additional networking and security considerations.

The following practices are applied:

- Use dedicated management and storage networks.
- Restrict access to the ESXi management interface.
- Use strong administrative credentials.
- Apply minimum necessary virtual switch security exceptions.
- Document Promiscuous Mode, MAC Address Changes, and Forged Transmits policies.
- Avoid exposing storage services to untrusted networks.
- Keep virtual appliances and guest operating systems updated.
- Monitor datastore capacity and virtual machine resource consumption.

For nested networking, some security policies may need to be relaxed to support traffic from nested virtual machines. These exceptions should be limited to the relevant lab switches and never assumed safe for production environments.

## 11. Project Deliverables

The completed project will include:

- VMware ESXi 8.0 standalone infrastructure.
- Two configured Standard Virtual Switches.
- Three Standard Port Groups.
- Management and storage VMkernel networking.
- OpenMediaVault storage integration.
- iSCSI and NFS datastore configuration.
- Debian Server deployment.
- Network and storage validation.
- Link-failure and recovery testing.
- Architecture diagrams and implementation documentation.
- Screenshots and troubleshooting evidence.

## 12. Project Scope and Limitations

This is an educational nested virtualization environment.

The project deliberately excludes features requiring vCenter Server, including:

- VMware Distributed Switch (vDS).
- vMotion.
- Distributed Resource Scheduler (DRS).
- VMware High Availability (HA) cluster management.

The infrastructure uses Standard Virtual Switches and the standalone ESXi Host Client.

Nested ESXi running under VMware Workstation is intended for learning, experimentation, and technical demonstrations, not production deployment.

## 13. Technologies and Tools

| Category           | Technology                   |
| ------------------ | ---------------------------- |
| Hypervisor         | VMware ESXi 8.0              |
| Virtualization     | VMware Workstation           |
| Virtual Networking | Standard vSwitch             |
| Network Services   | VMkernel                     |
| Storage            | iSCSI, NFS, VMFS 6           |
| NAS Platform       | OpenMediaVault               |
| Operating System   | Debian Server                |
| Network Testing    | ICMP, vmkping                |
| Administration     | ESXi Host Client, ESXi Shell |
| Documentation      | Markdown, Git, GitHub        |

## 14. Project Status

Current status: Infrastructure design and resource planning.

Implementation progress will be documented as each phase is completed.

| Phase                  | Status      |
| ---------------------- | ----------- |
| Architecture design    | Completed   |
| ESXi deployment        | Planned     |
| Local datastore        | Planned     |
| Virtual networking     | Planned     |
| VMkernel networking    | Planned     |
| Storage integration    | Planned     |
| Linux workloads        | Planned     |
| Infrastructure testing | Planned     |
| Documentation          | In progress |

## Author

Jeanluck NDATO\
Network & Cybersecurity Engineer

Interested in virtualization, network engineering, infrastructure security, and systems architecture.

- GitHub: [@jeanlucndato](https://github.com/jeanlucndato?utm_source=chatgpt.com)
- Technical Blog: [Geek Study](https://geekstudyn.blogspot.com?utm_source=chatgpt.com)

ESXI-IRONCORE — Building, testing, and understanding virtual infrastructure from the ground up.
