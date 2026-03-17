
# Nvidia Base Command Manager(BCM)

## Index

- [1. What is NVIDIA Base Command Manager](#1-what-is-nvidia-base-command-manager)
- [2. BCM Architecture](#2-bcm-architecture)
- [3. Node Provisioning Workflow](#3-node-provisioning-workflow)
- [4. Configuration Management in BCM](#4-configuration-management-in-bcm)
- [5. Cluster Operations with BCM](#5-cluster-operations-with-bcm)
- [6. Integration with HPC Ecosystem](#6-integration-with-hpc-ecosystem)
- [7. Production Troubleshooting](#7-production-troubleshooting)
- [8. Best Practices for Designing BCM Roles and Overlays](#8-best-practices-for-designing-bcm-roles-and-overlays)
- [9. Example: Managing a 128-GPU HPC Cluster with BCM](#9-example-managing-a-128-gpu-hpc-cluster-with-bcm)
- [10. How DevOps / SRE Engineers Use BCM](#10-how-devops--sre-engineers-use-bcm)


## 1. What is NVIDIA Base Command Manager

### 1.1 Overview

**NVIDIA Base Command Manager (BCM)** is a cluster management platform designed for **High Performance Computing (HPC)** and **AI infrastructure**. It provides tools to provision, configure, manage, and monitor large clusters of compute nodes from a centralized control plane.

In modern AI and HPC environments, clusters often contain:

- hundreds or thousands of compute nodes
- thousands of GPUs
- high-speed networking fabrics such as InfiniBand
- parallel storage systems

Managing such environments manually is extremely complex. BCM automates many of the operational tasks required to run these clusters efficiently.

BCM is commonly used in:

- AI training clusters
- research supercomputers
- enterprise HPC environments
- NVIDIA DGX SuperPOD deployments

The platform enables administrators to treat an entire cluster as a **single manageable system** rather than a collection of independent servers.

### 1.2 Why Cluster Management is Necessary

Traditional system administration methods work well for small environments but break down at HPC scale.

For example, imagine managing a cluster with:

|Resource|Quantity|
|---|---|
|Compute nodes|500|
|GPUs|4000|
|Network switches|dozens|
|Storage servers|multiple|

Without automation, administrators would need to manually:

- install operating systems on each node
- configure network settings
- install drivers and software packages
- maintain consistent configuration across nodes
- update firmware and BIOS settings
- monitor hardware health

Even small configuration mistakes could lead to cluster-wide failures.

Cluster management systems like BCM solve these problems by providing:

- automated provisioning
- centralized configuration management
- consistent software deployment
- scalable infrastructure management

### 1.3 Core Capabilities of BCM

BCM integrates several critical capabilities required to operate large HPC clusters.

#### 1.3.1 Node Provisioning

BCM automates node installation using **network boot (PXE)**.

When a new server is added to the cluster:

1. the node boots from the network
2. the operating system image is installed automatically
3. configuration roles are applied
4. services are started

This process allows administrators to deploy large clusters rapidly and consistently.

#### 1.3.2 Configuration Management

BCM maintains cluster configuration using a **hierarchical configuration model**.

Configuration is defined using several components:

- **software images** – base operating system environments
- **node categories** – grouping nodes with similar hardware
- **roles** – modular capabilities such as GPU drivers or Slurm clients
- **overlays** – bundles of roles applied to groups of nodes

This layered architecture allows flexible yet consistent configuration across the cluster.

#### 1.3.3 Workload Manager Integration

Most HPC clusters use a **workload scheduler** to allocate compute resources.

BCM commonly integrates with:

- Slurm
- PBS
- other HPC schedulers

The scheduler manages job execution on compute nodes, while BCM manages the underlying infrastructure.

For example:

1. users submit jobs to the scheduler
2. the scheduler selects available compute nodes
3. jobs are executed on nodes configured by BCM

#### 1.3.4 Hardware and Firmware Management

BCM can also manage hardware lifecycle operations, including:

- BIOS configuration
- firmware upgrades
- hardware monitoring
- remote power control

These tasks are performed through server management interfaces such as **IPMI** or **Redfish**.

This capability allows administrators to maintain consistent hardware configuration across the cluster.

#### 1.3.5 Monitoring and Health Management

BCM provides monitoring tools to track the health and performance of cluster components.

Metrics may include:

- node availability
- CPU and GPU utilization
- hardware failures
- storage usage
- network performance

Monitoring helps administrators detect issues quickly and maintain cluster reliability.

---

### 1.4 BCM Architecture

At a high level, BCM follows a centralized architecture.

```text
            +---------------------+
            |   BCM Head Node     |
            |  (Cluster Manager)  |
            +----------+----------+
                       |
        -------------------------------------
        |          |          |             |
   Compute Nodes  Login Nodes Storage Nodes Management Nodes
```

#### BCM Head Node

The head node is responsible for:

- cluster configuration database
- node provisioning services
- configuration management
- cluster monitoring

All cluster nodes communicate with the head node to obtain their configuration.

#### Managed Nodes

Managed nodes include:

- compute nodes running workloads
- login nodes used by users
- storage nodes providing shared data access

These nodes receive configuration instructions from the cluster manager.

### 1.5 BCM in AI and GPU Clusters

Modern HPC clusters increasingly focus on **GPU-accelerated computing**, especially for AI workloads.

GPU clusters require careful configuration of:

- NVIDIA drivers
- CUDA libraries
- high-speed networking (RDMA / InfiniBand)
- distributed training frameworks
- storage access patterns

BCM simplifies the deployment of GPU clusters by providing consistent configuration across nodes.

In environments such as **NVIDIA DGX SuperPOD**, BCM plays a central role in managing the entire infrastructure.

### 1.6 BCM vs Traditional Configuration Tools

BCM differs from common configuration tools such as Ansible or Puppet.

|Feature|BCM|Traditional Config Tools|
|---|---|---|
|OS provisioning|Yes|Usually external|
|Hardware management|Yes|Limited|
|HPC scheduler integration|Yes|No|
|Cluster topology awareness|Yes|No|
|Designed for HPC scale|Yes|Not specifically|

Traditional tools focus mainly on **software configuration**, while BCM manages the **entire cluster lifecycle**.

### 1.7 Benefits of Using BCM

Using BCM provides several advantages for HPC operations:

- **Automated deployment** of cluster nodes
- **Consistent configuration** across large environments
- **Simplified cluster scaling** when adding new nodes
- **Centralized management** of infrastructure components
- **Improved reliability** through standardized configuration

These benefits are particularly important in AI clusters where large training jobs depend on stable and predictable infrastructure.

### 1.8 Summary

NVIDIA Base Command Manager is a comprehensive platform for managing HPC and AI clusters. It provides automation and centralized control for provisioning, configuration, hardware management, and monitoring.

By combining these capabilities, BCM enables organizations to operate large-scale compute infrastructure efficiently while maintaining consistent configuration and high reliability.

Understanding the purpose and architecture of BCM is the foundation for learning how HPC clusters are deployed and managed in modern AI environments.

---

## 2. BCM Architecture

### 2.1 Introduction

NVIDIA Base Command Manager (BCM) is designed to manage large-scale HPC and AI clusters through a centralized architecture. Instead of managing each server individually, BCM provides a control plane that orchestrates provisioning, configuration, monitoring, and lifecycle management for all nodes in the cluster.

Understanding the architecture of BCM is important because it explains how configuration changes propagate across the cluster and how nodes interact with the cluster management system.

At a high level, BCM consists of:

- a **cluster manager (head node)**
- **provisioning services**
- **configuration management components**
- **managed nodes**
- **supporting infrastructure services**

These components work together to maintain a consistent and automated cluster environment.

### 2.2 High-Level Architecture

A typical BCM-managed cluster follows a centralized architecture.

```text
                    +------------------------+
                    |   BCM Head Node        |
                    |  (Cluster Manager)     |
                    +-----------+------------+
                                |
            ----------------------------------------
            |               |             |        |
        Login Nodes     Compute Nodes   Storage   Management
                                          Nodes       Nodes
```

The **BCM head node** acts as the central management point for the entire cluster, while the other nodes perform workload execution and data storage.

### 2.3 BCM Head Node

The BCM head node is the most important component of the cluster architecture. It hosts the services required to manage the entire cluster.

Key responsibilities include:

- cluster configuration database
- node provisioning services
- configuration management
- cluster monitoring
- integration with the workload scheduler

The head node typically runs several services that support these functions.

#### Key Services on the Head Node

|Service|Purpose|
|---|---|
|DHCP|assigns IP addresses during provisioning|
|TFTP|provides boot files for PXE boot|
|HTTP/Repository|distributes software images and packages|
|Configuration database|stores cluster configuration|
|Scheduler services|workload management integration|
|Monitoring services|collect cluster metrics|

Because the head node is critical for cluster operations, production environments may use **redundant management nodes** to improve reliability.

### 2.4 Node Types in a BCM Cluster

Clusters managed by BCM usually contain multiple types of nodes, each with a specific role.

#### 2.4.1 Compute Nodes

Compute nodes are the primary workers of the cluster. They execute HPC or AI workloads.

Characteristics of compute nodes:

- high CPU and GPU capacity
- high-speed network interfaces
- minimal user access
- optimized performance configuration

In AI clusters, compute nodes often contain multiple GPUs such as NVIDIA A100 or H100.

#### 2.4.2 Login Nodes

Login nodes provide user access to the cluster.

Users connect to login nodes through SSH to:

- submit jobs
- compile applications
- prepare datasets

Login nodes do not typically run heavy compute workloads.

#### 2.4.3 Storage Nodes

Storage nodes provide shared storage for the cluster.

Examples of storage services:

- NFS
- Lustre
- BeeGFS
- object storage gateways

Shared storage allows compute nodes to access the same datasets and user home directories.

### 2.4.4 Management Nodes

Management nodes provide operational services such as:

- monitoring
- logging
- metrics collection
- alerting systems

These services help administrators maintain cluster health.

### 2.5 Network Architecture

HPC clusters often use multiple network types to support different workloads.

### 2.5.1 Management Network

The management network is used for:

- node provisioning
- SSH access
- configuration management
- monitoring traffic

This network usually uses standard Ethernet.

#### 2.5.2 High-Speed Compute Network

Compute nodes often communicate through a high-performance network such as:

- InfiniBand
- RoCE (RDMA over Converged Ethernet)

This network provides:

- low latency
- high bandwidth
- direct memory access (RDMA)

These capabilities are essential for distributed computing and AI training workloads.

#### 2.5.3 Storage Network

Some clusters use a dedicated storage network for data access.

This separation helps prevent storage traffic from interfering with compute communication.

### 2.6 Provisioning Infrastructure

BCM automates node installation using network-based provisioning.

The provisioning process typically includes:

1. node boots via PXE
2. DHCP assigns an IP address
3. bootloader is downloaded from the head node
4. installation environment starts
5. software image is installed
6. configuration roles are applied

This system allows administrators to deploy hundreds of nodes automatically.

Provisioning services are typically hosted on the BCM head node.

### 2.7 Configuration Management Architecture

BCM configuration management relies on several components that work together to define the desired configuration of each node.

Key components include:

- software images
- node categories
- roles
- overlays

These components form a hierarchical configuration model that determines how nodes are configured.

For example:

```text
Software Image
       ↓
Node Category
       ↓
Roles
       ↓
Overlays
       ↓
Node-specific overrides
```

This layered architecture allows administrators to define default configurations while still allowing customization for specific nodes.

### 2.8 Integration with the Workload Scheduler

Most HPC clusters rely on a workload scheduler to allocate compute resources.

BCM commonly integrates with **Slurm**, which is one of the most widely used schedulers in HPC environments.

In this architecture:

1. BCM provisions and configures compute nodes
2. Slurm schedules jobs across those nodes
3. compute nodes execute jobs assigned by the scheduler

The scheduler relies on BCM to ensure that nodes are properly configured and available.

### 2.9 Monitoring and Observability

Maintaining cluster health requires visibility into system performance and hardware status.

BCM integrates monitoring tools that collect metrics such as:

- node health status
- CPU and GPU utilization
- memory usage
- network performance
- storage utilization

Monitoring systems help detect failures early and support proactive cluster management.

Many clusters integrate BCM monitoring with systems such as:

- Prometheus
- Grafana
- NVIDIA DCGM for GPU monitoring

### 2.10 Scalability Considerations

BCM is designed to manage large clusters efficiently.

Typical deployments may contain:

|Component|Example Scale|
|---|---|
|compute nodes|hundreds or thousands|
|GPUs|thousands|
|network switches|dozens|
|storage servers|multiple|

The centralized architecture allows administrators to apply configuration changes across many nodes simultaneously while maintaining consistent cluster state.

Automation and hierarchical configuration enable the system to scale without increasing operational complexity.

### 2.11 Summary

The architecture of NVIDIA Base Command Manager centers around a centralized cluster management system that coordinates provisioning, configuration management, monitoring, and integration with workload schedulers.

Key architectural components include:

- the BCM head node as the cluster control plane
- managed nodes performing compute, login, storage, and monitoring functions
- a layered configuration management system
- automated provisioning infrastructure
- integration with HPC networking and storage systems

This architecture allows large-scale HPC and AI clusters to be deployed and managed efficiently while maintaining consistent configuration and operational reliability.

---

## 3. Node Provisioning Workflow

### 3.1 Introduction

Node provisioning is the process of automatically installing and configuring a server so that it becomes a functional member of the cluster. In NVIDIA Base Command Manager (BCM), provisioning allows administrators to deploy large numbers of nodes quickly and consistently.

Instead of manually installing operating systems and configuring services on each machine, BCM uses an automated workflow that includes:

- network boot
- operating system installation
- configuration management
- role and overlay application

This automated provisioning process ensures that every node in the cluster follows the same configuration standards.

### 3.2 Overview of the Provisioning Workflow

The node provisioning workflow in BCM generally follows these stages:

```text
Node Power On
     ↓
PXE Boot
     ↓
DHCP Assignment
     ↓
Bootloader Download (TFTP)
     ↓
Provisioning Environment
     ↓
Operating System Installation
     ↓
First Boot
     ↓
BCM Configuration Application
     ↓
Node Ready for Workloads
```

Each stage plays an important role in transforming a bare-metal server into a fully configured cluster node.

---

### 3.3 Step 1: Node Power-On

Provisioning begins when a server node is powered on. This may occur when:

- a new node is added to the cluster
- an existing node is reinstalled
- hardware is replaced
- a node is reprovisioned for a different category

The node typically boots using **PXE (Preboot Execution Environment)** so that it can obtain its operating system from the network.

### 3.4 Step 2: PXE Network Boot

PXE allows a system without an installed operating system to boot from a network server.

During PXE boot, the node’s network interface card performs the following actions:

1. broadcasts a DHCP discovery request
2. waits for a DHCP response from the provisioning server
3. downloads bootloader information

Example broadcast request:

```text
DHCPDISCOVER
```

This request asks the network if any provisioning server can provide boot instructions.

### 3.5 Step 3: DHCP Assignment

The BCM head node typically runs a DHCP service responsible for assigning IP addresses to nodes during provisioning.

The DHCP response contains several important pieces of information:

- an IP address for the node
- the location of the boot server
- the name of the bootloader file

Example response:

```text
IP Address: 10.10.1.15
Boot Server: 10.10.1.1
Boot File: pxelinux.0
```

After receiving this information, the node knows where to obtain the bootloader.

### 3.6 Step 4: Bootloader Download (TFTP)

Once the DHCP process completes, the node downloads the bootloader using **TFTP (Trivial File Transfer Protocol)**.

Typical files downloaded include:

```text
pxelinux.0
vmlinuz
initrd.img
```

These files are hosted on the BCM head node.

The bootloader then launches a temporary operating system used for provisioning.

### 3.7 Step 5: Provisioning Environment

The node now boots into a minimal Linux environment known as the **provisioning environment**.

This environment performs several tasks:

- detects hardware components
- establishes communication with the BCM cluster manager
- retrieves provisioning instructions
- prepares disks for installation

The provisioning environment determines the identity of the node using attributes such as:

- MAC address
- hostname
- node definition in the BCM configuration database

Example node definition:

```text
node: compute03
MAC: 3C:EC:EF:11:22:33
category: compute-gpu
image: gpu-baseos
```

This information tells BCM how the node should be configured.

### 3.8 Step 6: Disk Partitioning

Before installing the operating system, the provisioning system prepares the node’s disks.

Typical partition layout for HPC compute nodes may include:

```text
/boot
/
/var
/tmp
```

Some systems may also include high-performance local storage for temporary data:

```text
/scratch
```

Disk partitioning rules are usually defined within the **software image configuration**.

### 3.9 Step 7: Operating System Installation

Once the disk layout is prepared, the node installs the operating system from the assigned software image.

In BCM, a software image is often stored as a **filesystem tree** containing the base operating system and essential packages.

Example structure:

```text
gpu-baseos/
 ├── bin
 ├── etc
 ├── lib
 ├── usr
```

During installation, these files are copied to the node’s root filesystem.

After the installation completes, the node is ready for its first full system boot.

### 3.10 Step 8: First Boot

After the operating system is installed, the node reboots and starts from its local disk.

At this stage, the node runs the base operating system but has not yet received its full configuration.

During the first boot:

- network interfaces are configured
- system services start
- the BCM management agent initializes

The node then contacts the BCM head node to obtain its configuration.

### 3.11 Step 9: Configuration Application

After the node registers with the BCM controller, configuration management begins.

BCM determines the node’s configuration based on:

- software image
- node category
- assigned roles
- applied overlays

The system then performs several configuration tasks.

#### Package Installation

Required packages are installed automatically.

Example:

```text
dnf install slurm munge
dnf install nvidia-driver
```

#### Configuration File Generation

Templates are rendered to generate configuration files.

Example file:

```text
/etc/slurm/slurm.conf
```

#### Storage Configuration

Shared storage mounts are configured.

Example:

```text
nfs-server:/home /home nfs defaults 0 0
```

#### Service Management

Required services are enabled and started.

Example:

```text
systemctl enable slurmd
systemctl start slurmd
```

After these steps complete, the node reaches its desired configuration state.

### 3.12 Step 10: Node Ready for Workloads

Once provisioning and configuration are complete, the node becomes available for workload execution.

If the cluster uses the Slurm scheduler, the node will appear in the scheduler’s node list.

Example:

```bash
sinfo
```

Output:

```text
NODELIST   STATE
compute01  idle
compute02  idle
compute03  idle
```

At this point, the node is fully integrated into the cluster and ready to execute jobs.

### 3.13 Reprovisioning Nodes

BCM also supports **reprovisioning**, which allows administrators to reinstall nodes automatically.

Common scenarios for reprovisioning include:

- software upgrades
- configuration changes
- hardware replacement
- testing new cluster configurations

Reprovisioning ensures that nodes can be rebuilt quickly while maintaining configuration consistency.

## 3.14 Benefits of Automated Provisioning

Automated provisioning provides several advantages for HPC clusters.

|Benefit|Description|
|---|---|
|Consistency|every node receives the same configuration|
|Speed|large clusters can be deployed quickly|
|Scalability|hundreds of nodes can be provisioned simultaneously|
|Reliability|reduces manual configuration errors|
|Reproducibility|clusters can be rebuilt when necessary|

These capabilities are essential for operating modern AI and HPC environments.

## 3.15 Summary

Node provisioning in BCM automates the process of installing and configuring cluster nodes using a network-based workflow.

The provisioning pipeline includes:

- PXE network boot
- operating system installation
- configuration management
- role and overlay application

This automated process allows administrators to deploy and manage large HPC clusters efficiently while ensuring consistent configuration across all nodes.

Understanding the provisioning workflow is fundamental for operating and troubleshooting BCM-managed clusters.


---
## 4. Configuration Management in BCM


Configuration management is one of the core capabilities of **NVIDIA Base Command Manager (BCM)**. In HPC environments where clusters may contain hundreds or thousands of nodes, manual configuration is impractical and error-prone. BCM provides a structured configuration management framework that ensures **consistent, reproducible, and scalable cluster operations**.

BCM uses a **hierarchical configuration model** combined with **roles, categories, overlays, and software images** to define and enforce the desired configuration state of every node in the cluster.

The main goal is to guarantee that all nodes are configured correctly while allowing flexible customization for different node types and workloads.

### 4.1. Key Concepts

#### 4.1.1 Cluster

A **cluster** is the top-level object in BCM. It represents the entire managed HPC environment, including:

- Head nodes
- Compute nodes
- Login nodes
- Storage nodes
- Networking devices

The cluster manager maintains the configuration database and orchestrates provisioning, updates, and monitoring.

#### 4.1.2 Node

A **node** represents a physical server in the cluster. Nodes can perform different functions such as:

- Compute node
- Login node
- Storage node
- Management node

Each node receives its configuration from the BCM configuration hierarchy.

#### 4.1.3 Software Image

A **software image** defines the base operating system environment used during node provisioning.

In BCM, a software image is typically a **filesystem tree** that contains:

- OS packages
- system libraries
- default configuration files

The image serves as the baseline for node installation.

Example:

```
gpu-baseos
 ├── /bin
 ├── /etc
 ├── /usr
 └── /lib
```

Images are usually kept minimal, with additional functionality applied later through roles.

#### 4.1.4 Node Category

A **node category** groups nodes with similar hardware or functional characteristics.

Examples:

- compute-gpu
- login-node
- storage-node
- head-node

Categories define:

- default software image
- baseline configuration parameters
- hardware tuning settings

Each node belongs to **exactly one category**.

#### 4.1.4 Role

A **role** represents a specific capability or function that can be applied to a node.

Roles typically include:

- package installation
- configuration templates
- scripts
- service management

Examples of roles:

|Role|Purpose|
|---|---|
|slurm-client|enables job execution on compute nodes|
|slurm-controller|runs the Slurm scheduler|
|gpu-driver|installs NVIDIA GPU drivers|
|nfs-client|mounts shared storage|
|monitoring-agent|exports metrics|

Roles are designed to be **modular and reusable building blocks**.

#### 4.1.6 Overlay

An **overlay** is a mechanism used to apply a group of roles to a set of nodes.

Overlays simplify cluster management by bundling related roles together.

Example:

```
compute-overlay
 ├── slurm-client
 ├── gpu-driver
 ├── nfs-client
 └── monitoring-agent
```

Nodes assigned to this overlay automatically receive all roles defined within it.

#### 4.1.7 Overlay Priority

When multiple overlays apply to a node, **priority determines which configuration takes precedence** if conflicts occur.

Higher priority overlays override lower priority ones.

Example priority structure:

|Overlay|Priority|Purpose|
|---|---|---|
|base-overlay|100|baseline configuration|
|compute-overlay|200|compute node functionality|
|performance-overlay|400|HPC performance tuning|
|debug-overlay|800|temporary debugging|

Overlay priority allows flexible layering of configuration changes.

### 4.2. Hierarchical Configuration Model

BCM uses a hierarchical configuration model where configuration is applied in layers.

The typical order of configuration evaluation is:

```
Software Image
      ↓
Node Category
      ↓
Roles
      ↓
Overlays (priority order)
      ↓
Node-specific overrides
```

Higher layers override values defined in lower layers.

This model allows administrators to define default settings at higher levels while allowing specific customization when needed.

### 4.3 Configuration Rendering

Configuration files are usually generated from **templates**.

Templates contain variables that BCM replaces with actual values when generating node configuration.

Example template:

```
ControlMachine={{ slurm_controller }}
NodeName={{ node_name }} CPUs={{ cpu_count }}
```

After rendering:

```
ControlMachine=head01
NodeName=compute01 CPUs=128
```

This approach ensures configuration consistency across nodes.

### 4.4 Configuration Lifecycle

BCM enforces configuration through a lifecycle process.

#### Step 1: Node Provisioning

Nodes are installed using PXE network boot and receive their base operating system from the assigned software image.

#### Step 2: Role Application

Roles install required packages, generate configuration files, and start services.

#### Step 3: Overlay Application

Overlays apply additional role bundles and configuration adjustments.

#### Step 4: Configuration Convergence

BCM ensures that the node matches the desired configuration state. If configuration drift occurs, the system can reapply the correct configuration.

### 4.5. Example: GPU Compute Node Configuration

Consider a GPU compute node.

Category:

```
compute-gpu
```

Overlays applied:

```
base-overlay
slurm-overlay
compute-overlay
performance-overlay
```

Roles resulting from overlays:

```
base-os
time-sync
monitoring-agent
munge
slurm-config
slurm-client
gpu-driver
cuda-runtime
nfs-client
dcgm-agent
rdma-stack
nccl-tuning
```

This node becomes capable of:

- executing Slurm jobs
- running GPU workloads
- accessing shared storage
- exporting monitoring metrics
- participating in distributed training

### 4.6. Best Practices for BCM Configuration Management

#### 4.6.1 Use Modular Roles

Roles should represent a single capability and be reusable across node types.

#### 4.6.2 Avoid Role Explosion

Do not create many specialized roles. Instead, use overlays and parameters to customize behavior.

#### 4.6.3 Follow the Single Writer Rule

Each configuration file should have only one role responsible for managing it.

#### 4.6.4 Keep Software Images Minimal

The base image should contain only the operating system and essential components. Additional software should be installed through roles.

#### 4.6.5 Use Overlays for Environment Differences

Different environments (production, testing, debugging) should be controlled using overlays rather than modifying core roles.

### 4.7. Benefits of BCM Configuration Management

BCM configuration management provides several advantages for HPC operations:

- **Consistency** across hundreds or thousands of nodes
- **Reproducibility** of cluster deployments
- **Scalability** for large GPU clusters
- **Automation** of provisioning and updates
- **Centralized management** of configuration policies

These capabilities are critical for modern HPC and AI clusters where workloads require predictable and high-performance infrastructure.

### 4.8. Conclusion

Configuration management in BCM is built around a hierarchical and modular architecture. By combining software images, categories, roles, and overlays, BCM enables administrators to manage complex HPC clusters in a consistent and scalable way.

Proper use of BCM configuration management practices allows organizations to operate large-scale AI infrastructure efficiently while minimizing operational risk.


---

## 5. Cluster Operations with BCM

### 5.1 Introduction

Operating an HPC cluster involves far more than simply installing nodes. Administrators must continuously manage node health, update software, maintain configuration consistency, monitor system performance, and respond to operational incidents.

NVIDIA Base Command Manager (BCM) provides a centralized platform that simplifies these operational tasks. By combining provisioning, configuration management, hardware control, and monitoring capabilities, BCM enables administrators to operate large clusters efficiently.

Typical cluster operations include:

- managing node lifecycle
- updating software and drivers
- monitoring cluster health
- handling node failures
- performing maintenance and upgrades

Understanding how these operational workflows function is essential for maintaining a reliable HPC environment.

### 5.2 Node Lifecycle Management

Nodes in an HPC cluster typically move through several operational states during their lifecycle.

Common node states include:

|State|Description|
|---|---|
|Provisioning|node is being installed|
|Active|node is available for workloads|
|Draining|node is finishing current jobs before maintenance|
|Maintenance|node temporarily removed from scheduling|
|Offline|node not available due to failure|

BCM allows administrators to manage these states while coordinating with the workload scheduler.

For example, before performing maintenance on a compute node, the node can be drained from the scheduler.

Example using Slurm:

```bash
scontrol update NodeName=compute03 State=DRAIN
```

After maintenance is complete, the node can be returned to service:

```bash
scontrol update NodeName=compute03 State=RESUME
```

This workflow prevents interruption of running jobs.

### 5.3 Adding New Nodes to the Cluster

Clusters often grow as additional compute resources are added.

The typical process for adding new nodes includes:

1. physically installing the hardware
2. connecting network and power
3. registering the node in BCM
4. assigning a node category
5. provisioning the node through PXE installation

Once provisioning is complete, the node automatically receives configuration roles and joins the cluster.

Because configuration is centrally managed, new nodes become operational with minimal manual work.

### 5.4 Software and Driver Updates

Cluster software components require periodic updates to maintain security, compatibility, and performance.

Common updates include:

- operating system packages
- GPU drivers
- CUDA libraries
- scheduler software
- container runtimes

BCM allows administrators to update these components through roles and software images.

Typical update workflow:

1. update the role or software image
2. apply the updated configuration to selected nodes
3. restart affected services
4. verify system functionality

Updates are often applied gradually to avoid disrupting workloads.

### 5.5 Rolling Maintenance

In production clusters, updates and maintenance must be performed without affecting active workloads.

A common strategy is **rolling maintenance**, where nodes are updated one at a time.

Example workflow:

1. drain a compute node from the scheduler
2. apply configuration updates
3. reboot the node if required
4. validate node functionality
5. return node to service
6. repeat for the next node

This approach allows the cluster to remain operational while updates are performed.

### 5.6 Monitoring and Observability

Maintaining cluster health requires continuous monitoring of system resources and hardware components.

BCM can integrate with monitoring tools to collect metrics such as:

- CPU usage
- GPU utilization
- memory consumption
- network performance
- storage throughput
- node availability

Monitoring data helps administrators identify performance bottlenecks and hardware failures.

Many clusters integrate BCM monitoring with systems such as:

- Prometheus
- Grafana
- NVIDIA DCGM for GPU metrics

These tools provide dashboards and alerting capabilities for cluster administrators.

### 5.7 Handling Node Failures

Hardware failures are inevitable in large clusters. BCM helps administrators quickly identify and isolate faulty nodes.

Common failure scenarios include:

- GPU hardware errors
- disk failures
- network connectivity problems
- power supply issues
- memory errors

When a failure occurs, the affected node can be removed from scheduling to prevent further job failures.

Example:

```bash
scontrol update NodeName=compute05 State=DRAIN Reason="hardware issue"
```

Administrators can then investigate the issue using system logs and hardware monitoring tools.

Once the problem is resolved, the node can be re-enabled.

### 5.8 Configuration Drift Management

Over time, nodes may drift from their intended configuration due to manual changes or unexpected system behavior.

Examples of configuration drift include:

- modified configuration files
- incorrect package versions
- disabled services
- inconsistent system settings

BCM helps prevent configuration drift by maintaining a centralized configuration model.

If a node deviates from its desired configuration, BCM can reapply the correct roles and templates to restore the expected state.

This capability ensures that cluster nodes remain consistent over time.

### 5.9 Hardware and Firmware Maintenance

Cluster operations also include maintaining hardware components.

Examples of hardware maintenance tasks include:

- updating BIOS firmware
- upgrading network adapter firmware
- replacing failing disks
- updating GPU firmware

BCM can coordinate these updates across multiple nodes using automated workflows.

Maintenance operations are typically scheduled during low utilization periods to minimize disruption.

### 5.10 Backup and Disaster Recovery

Reliable cluster operations require backup strategies for critical system components.

Important items to protect include:

- BCM configuration database
- scheduler configuration
- user data stored on shared storage
- monitoring configurations

Regular backups allow administrators to restore the cluster environment in case of system failure or data corruption.

Disaster recovery procedures should be tested periodically to ensure they function correctly.

### 5.11 Operational Best Practices

Successful cluster operations depend on following several best practices.

#### Standardize Configuration

All nodes should follow consistent configuration standards using roles and overlays.

#### Use Automation

Manual configuration changes should be avoided whenever possible. Automation reduces human error.

#### Monitor System Health

Continuous monitoring allows administrators to detect issues early.

#### Perform Regular Maintenance

Software updates and hardware checks should be scheduled regularly to prevent unexpected failures.

#### Document Operational Procedures

Clear documentation ensures that administrators can respond quickly to incidents and perform maintenance tasks consistently.

### 5.12 Summary

Cluster operations involve managing the lifecycle, configuration, health, and performance of all nodes within the HPC environment.

BCM provides a centralized platform that simplifies these operational tasks by enabling:

- automated provisioning and configuration
- consistent software management
- coordinated maintenance workflows
- comprehensive monitoring and health management

By combining automation with centralized management, BCM allows administrators to operate large HPC clusters efficiently while maintaining high availability and system reliability.

---

## 6. Integration with HPC Ecosystem

### 6.1 Introduction

High Performance Computing (HPC) environments consist of multiple specialized components working together to support large-scale scientific and AI workloads. These components typically include workload schedulers, high-speed networking fabrics, parallel storage systems, container platforms, and monitoring frameworks.

NVIDIA Base Command Manager (BCM) is designed to operate as the **infrastructure management layer** within this ecosystem. It integrates with various HPC technologies to provide a unified platform for provisioning, configuration, and lifecycle management of cluster nodes.

Rather than replacing other HPC tools, BCM works alongside them to ensure that all cluster components operate together reliably and consistently.

### 6.2 Workload Scheduler Integration

Most HPC clusters rely on a **workload scheduler** to manage compute resources and allocate them to user jobs.

Common schedulers used in HPC environments include:

|Scheduler|Description|
|---|---|
|Slurm|widely used open-source scheduler|
|PBS Pro|commercial workload manager|
|Torque|open-source PBS derivative|

Among these options, **Slurm** is the most commonly used scheduler in modern AI and HPC clusters.

#### Role of the Scheduler

Schedulers perform several key tasks:

- queueing user jobs
- allocating compute nodes
- enforcing resource limits
- managing job priorities
- scheduling workloads efficiently

In a BCM-managed cluster, the scheduler relies on BCM to ensure that nodes are correctly configured and available for scheduling.

Typical interaction workflow:

```text
User submits job
      ↓
Scheduler selects nodes
      ↓
Compute nodes execute job
      ↓
Results stored on shared storage
```

BCM ensures that compute nodes have the required software and configuration to run scheduled workloads.

### 6.3 GPU Computing Ecosystem

Modern HPC clusters increasingly rely on GPUs for accelerating AI training, scientific simulations, and data processing workloads.

Key components of the GPU ecosystem include:

| Component      | Purpose                         |
| -------------- | ------------------------------- |
| NVIDIA Drivers | enable GPU hardware access      |
| CUDA           | GPU programming platform        |
| NCCL           | multi-GPU communication library |
| cuDNN          | deep learning acceleration      |
| DCGM           | GPU monitoring and management   |

BCM simplifies GPU cluster management by ensuring that:

- GPU drivers are consistently installed across nodes
- CUDA libraries are available to applications
- GPU configuration parameters are applied automatically
- monitoring tools collect GPU performance metrics

This consistency is critical for distributed training workloads where software mismatches can cause failures.

### 6.4 High-Speed Networking Integration

Distributed HPC workloads require extremely fast communication between nodes. Standard Ethernet networks are often insufficient for these workloads.

Many HPC clusters use high-performance networking technologies such as:

- InfiniBand
- RDMA over Converged Ethernet (RoCE)

These networks provide:

- low latency communication
- high bandwidth data transfer
- direct memory access between nodes

BCM helps configure these networks by managing:

- network interface settings
- RDMA drivers
- performance tuning parameters

Proper network configuration is essential for distributed training frameworks that rely on fast inter-node communication.

### 6.5 Storage System Integration

Large-scale HPC workloads require shared storage systems that allow compute nodes to access common datasets.

Common storage technologies include:

|Storage System|Description|
|---|---|
|NFS|simple shared filesystem|
|Lustre|parallel filesystem for HPC|
|BeeGFS|high-performance distributed storage|
|Object storage|scalable data storage|

Shared storage is used for several purposes:

- user home directories
- shared software environments
- training datasets
- simulation output data

BCM integrates with these storage systems by configuring:

- mount points
- client software
- authentication settings

This ensures that all compute nodes can access shared data consistently.

### 6.6 Container Platforms

Container technologies are widely used in HPC and AI environments to package applications and dependencies.

Popular container systems include:

|Container Platform|Description|
|---|---|
|Docker|widely used container runtime|
|Podman|daemonless container engine|
|Singularity / Apptainer|HPC-focused container system|
|Kubernetes|container orchestration platform|

Containers allow users to run applications in isolated environments without affecting the base system configuration.

#### Traditional HPC Containers

Historically, HPC environments favored container systems such as **Singularity (Apptainer)** because they integrate well with shared filesystems and HPC schedulers like Slurm.

These systems allow users to run containerized workloads while maintaining security and compatibility with multi-user HPC clusters.

BCM can deploy container runtimes across cluster nodes through configuration roles, ensuring that containerized workloads can run consistently across the cluster.

#### Kubernetes in HPC Environments

In recent years, **Kubernetes** has become increasingly popular in AI infrastructure. Kubernetes provides advanced container orchestration capabilities including:

- automated workload scheduling
- container lifecycle management
- resource allocation
- service discovery
- horizontal scaling

Many modern AI clusters run Kubernetes alongside traditional HPC schedulers.

Typical hybrid architectures include:

|Model|Description|
|---|---|
|Slurm-only|traditional HPC workloads|
|Kubernetes-only|cloud-native AI workloads|
|Slurm + Kubernetes|hybrid HPC/AI cluster|

In these environments:

- BCM manages **node provisioning and configuration**
- Kubernetes manages **containerized workloads**

BCM ensures that nodes are prepared with the required components for Kubernetes operation, such as:

- container runtime
- GPU drivers
- NVIDIA GPU Operator
- network configuration
- storage mounts

This integration enables Kubernetes to schedule GPU workloads efficiently on cluster nodes.

### 6.7 Monitoring and Observability Tools

Monitoring is essential for maintaining the health and performance of large HPC clusters.

Typical monitoring tools used in HPC environments include:

|Tool|Purpose|
|---|---|
|Prometheus|metrics collection|
|Grafana|visualization dashboards|
|NVIDIA DCGM|GPU monitoring|
|Node Exporter|system metrics|

These tools collect information such as:

- CPU utilization
- GPU utilization
- memory usage
- network throughput
- storage performance

BCM integrates with monitoring systems by deploying monitoring agents on cluster nodes and ensuring consistent configuration.

Administrators can then visualize cluster performance through dashboards and alerts.

### 6.8 AI Framework Integration

Many modern HPC clusters support large-scale AI training workloads using distributed machine learning frameworks.

Common frameworks include:

|Framework|Purpose|
|---|---|
|PyTorch|deep learning framework|
|TensorFlow|machine learning platform|
|Horovod|distributed training library|
|DeepSpeed|optimized large model training|

These frameworks rely on several underlying technologies:

- GPUs for acceleration
- NCCL for communication
- high-speed networking
- shared datasets

BCM ensures that compute nodes have the necessary runtime environments to support these frameworks.

### 6.9 Cloud and Hybrid HPC Integration

Some organizations operate hybrid environments where HPC clusters interact with cloud resources.

Examples include:

- bursting workloads to cloud GPUs
- storing datasets in cloud object storage
- running hybrid workflows across on-premises and cloud systems

BCM can support hybrid environments by managing the on-premises cluster infrastructure while integrating with external services.

This allows organizations to scale compute resources beyond their local cluster when necessary.

## 6.10 Benefits of Ecosystem Integration

Integration with the broader HPC ecosystem provides several benefits:

|Benefit|Description|
|---|---|
|flexibility|supports diverse workloads|
|scalability|enables large distributed jobs|
|reliability|ensures consistent configuration|
|performance|optimized hardware utilization|
|interoperability|works with standard HPC tools|

These integrations allow BCM-managed clusters to support a wide range of scientific, engineering, and AI workloads.

## 6.11 Summary

BCM operates as the infrastructure management layer within the HPC ecosystem. It integrates with multiple technologies including workload schedulers, GPU computing platforms, high-speed networking systems, storage solutions, container runtimes, and monitoring frameworks.

By coordinating these components, BCM enables administrators to deploy and manage complex HPC clusters efficiently while ensuring that all systems work together seamlessly.

Understanding how BCM interacts with the broader HPC ecosystem is essential for designing and operating modern HPC and AI infrastructure.

---

## 7. Production Troubleshooting

### 7.1 Introduction

Operating an HPC cluster in production inevitably involves diagnosing and resolving operational issues. In large clusters containing hundreds or thousands of nodes, failures can occur at many layers of the system, including hardware, networking, storage, configuration management, and workload scheduling.

NVIDIA Base Command Manager (BCM) provides centralized tools that help administrators troubleshoot problems efficiently. However, effective troubleshooting still requires a systematic approach and an understanding of how different components of the HPC ecosystem interact.

This chapter describes common troubleshooting practices used in production clusters and provides a structured approach to diagnosing problems.

### 7.2 Troubleshooting Methodology

When troubleshooting HPC systems, administrators typically follow a layered approach that moves from high-level symptoms to root causes.

A common troubleshooting workflow includes:

```text
Identify the problem
       ↓
Check scheduler state
       ↓
Check node health
       ↓
Verify GPU status
       ↓
Verify network connectivity
       ↓
Check storage performance
       ↓
Inspect system logs
```

This structured approach helps isolate the component responsible for the issue.

### 7.3 Scheduler Issues

The workload scheduler is often the first place where cluster problems become visible.

#### Common Symptoms

- jobs remain in pending state
- nodes appear unavailable
- jobs fail immediately after submission

Example command:

```bash
squeue
```

Example output:

```text
JOBID PARTITION NAME USER ST TIME NODES NODELIST(REASON)
1245  compute   train user PD 0:00 4 (Nodes unavailable)
```

#### Diagnostic Steps

1. Check node availability:

```bash
sinfo
```

2. Inspect node status:

```bash
scontrol show node compute03
```

3. Verify scheduler services:

```bash
systemctl status slurmctld
systemctl status slurmd
```

#### Typical Causes

- scheduler daemon failure
- node configuration mismatch
- node health check failure
- authentication issues (munge)

### 7.4 Node Configuration Problems

Nodes may fail to operate correctly if configuration roles or overlays are applied incorrectly.

#### Symptoms

- services fail to start
- missing configuration files
- incorrect software versions

#### Diagnostic Steps

Check active services:

```bash
systemctl status slurmd
```

Verify installed packages:

```bash
rpm -qa | grep slurm
```

Inspect configuration files:

```bash
cat /etc/slurm/slurm.conf
```

#### Possible Causes

- role misconfiguration
- template rendering errors
- configuration conflicts between overlays

Ensuring that configuration files are owned by a single role can help prevent these issues.

### 7.5 GPU Runtime Issues

GPU failures can significantly affect AI workloads.

#### Symptoms

- GPUs not detected
- training jobs fail
- reduced GPU utilization

Check GPU status:

```bash
nvidia-smi
```

Example failure:

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver
```

#### Diagnostic Steps

Check kernel modules:

```bash
lsmod | grep nvidia
```

Check hardware detection:

```bash
lspci | grep -i nvidia
```

Check kernel logs:

```bash
dmesg | grep NVRM
```

### Common Causes

- GPU driver mismatch
- kernel upgrade without driver rebuild
- hardware failure
- incorrect CUDA installation

### 7.6 Networking Issues

Distributed workloads depend heavily on reliable networking.

#### Symptoms

- slow distributed training
- communication timeouts
- failed multi-node jobs

Check basic connectivity:

```bash
ping compute02
```

Check network interfaces:

```bash
ip addr
```

For InfiniBand clusters:

```bash
ibstat
```

#### Diagnostic Steps

Verify interface status:

```bash
ethtool eth0
```

Check RDMA configuration:

```bash
rdma link
```

Investigate switch or cable failures if connectivity is unstable.

### 7.7 Storage Performance Problems

Storage bottlenecks are common in AI training workloads where large datasets are accessed by many nodes simultaneously.

#### Symptoms

- slow job startup
- high I/O wait
- GPU utilization drops

Check disk I/O performance:

```bash
iostat -x 1
```

Check filesystem mounts:

```bash
mount
```

Check NFS statistics:

```bash
nfsstat
```

#### Common Causes

- overloaded NFS server
- metadata bottleneck in parallel filesystem
- network congestion affecting storage traffic

Proper storage architecture and monitoring can help mitigate these problems.

### 7.8 Provisioning Failures

Node provisioning failures may occur when installing or rebuilding nodes.

#### Symptoms

- node stuck at PXE boot
- operating system installation fails
- node does not appear in BCM

#### Diagnostic Steps

Check DHCP logs on the head node:

```bash
journalctl -u dhcpd
```

Check TFTP service:

```bash
journalctl -u tftp
```

Verify network connectivity and PXE configuration.

Provisioning problems often occur due to incorrect MAC addresses or network configuration errors.

### 7.9 Hardware Failures

Large clusters inevitably experience hardware failures.

Common hardware problems include:

- GPU failures
- memory errors
- disk failures
- network adapter issues

Monitoring systems usually detect these failures early.

Example hardware inspection:

```bash
dmesg | grep error
```

Check SMART disk health:

```bash
smartctl -a /dev/sda
```

Check GPU health:

```bash
nvidia-smi -q
```

Faulty hardware should be isolated quickly to prevent job failures.

### 7.10 Log Analysis

Logs provide critical information during troubleshooting.

Important log locations include:

|Component|Log Location|
|---|---|
|system services|journalctl|
|scheduler|/var/log/slurm|
|GPU drivers|kernel logs|
|provisioning|BCM logs|

Example command:

```bash
journalctl -xe
```

Careful log analysis often reveals configuration or hardware problems.

### 7.11 Preventive Troubleshooting Practices

Effective troubleshooting also involves preventing problems before they occur.

Recommended practices include:

- continuous monitoring of cluster health
- automated alerting systems
- periodic hardware diagnostics
- configuration drift detection
- regular system updates

These practices help maintain cluster stability and reduce downtime.

### 7.12 Summary

Production troubleshooting in HPC clusters requires a systematic approach that considers all components of the system.

Key troubleshooting areas include:

- scheduler operation
- node configuration
- GPU runtime environment
- networking infrastructure
- storage performance
- provisioning workflow
- hardware health

By following structured diagnostic procedures and using BCM’s centralized management capabilities, administrators can identify and resolve issues efficiently, ensuring reliable operation of the HPC cluster.

---

## 8. Best Practices for Designing BCM Roles and Overlays

### 8.1 Introduction

Roles and overlays are the core building blocks of configuration management in NVIDIA Base Command Manager (BCM). A well-designed role and overlay structure enables scalable, maintainable, and reliable cluster operations. Poor design, however, can lead to configuration conflicts, operational complexity, and production outages.

This chapter outlines best practices for designing roles and overlays in BCM, focusing on clarity, modularity, and scalability.

### 8.2 Design Principles

Effective BCM design follows several key principles:

- **Modularity**: roles should represent a single capability
- **Reusability**: roles should be applicable across multiple node types
- **Separation of concerns**: avoid mixing unrelated functionality
- **Consistency**: enforce standard patterns across the cluster
- **Simplicity**: avoid unnecessary complexity

These principles help maintain a clean configuration structure as the cluster grows.

### 8.3 Designing Roles

#### 8.3.1 Roles Represent Capabilities

Each role should define a single capability.

Examples:

|Role|Capability|
|---|---|
|gpu-driver|installs NVIDIA driver|
|slurm-client|enables job execution|
|nfs-client|mounts shared storage|
|monitoring-agent|exports metrics|

Roles should answer the question:

> What functionality does this role provide?

If a role contains multiple unrelated functions, it should be split.

#### 8.3.2 Avoid Role Explosion

A common mistake is creating too many specialized roles.

Example of poor design:

```text
gpu-driver-h100
gpu-driver-a100
gpu-driver-test
compute-node-role-v2
```

This leads to:

- configuration duplication
- maintenance difficulty
- inconsistent environments

Instead, use **parameterized roles**.

Example:

```text
gpu-driver
```

With parameters:

```text
driver_version = 535
```

Different categories or overlays can set different values without creating new roles.

#### 8.3.3 Follow the Single Writer Rule

Each configuration file should be managed by only one role.

Example:

|File|Owner Role|
|---|---|
|/etc/slurm/slurm.conf|slurm-config|
|/etc/fstab|nfs-client|
|/etc/docker/daemon.json|container-runtime|

If multiple roles modify the same file, conflicts may occur and lead to unpredictable behavior.

#### 8.3.4 Separate Configuration and Service Roles

Roles can be separated into:

- configuration roles (generate files)
- service roles (run services)

Example:

|Role|Responsibility|
|---|---|
|slurm-config|generates slurm.conf|
|slurm-client|runs slurmd|
|slurm-controller|runs slurmctld|

This separation improves flexibility and avoids unintended service activation.

#### 8.3.5 Keep Roles Idempotent

Roles should be safe to apply multiple times without causing inconsistent states.

Examples:

- package installation should not reinstall unnecessarily
- configuration files should be deterministic
- services should be enabled consistently

Idempotent roles ensure reliable configuration convergence.

### 8.4 Designing Overlays

#### 8.4.1 Use Overlays for Composition

Overlays group roles into functional sets.

Example:

```text
compute-overlay
 ├── slurm-client
 ├── gpu-driver
 ├── nfs-client
 └── monitoring-agent
```

This allows administrators to apply multiple capabilities at once.

#### 8.4.2 Keep Overlays Simple

Each overlay should have a clear purpose.

Examples:

|Overlay|Purpose|
|---|---|
|base-overlay|baseline system configuration|
|compute-overlay|compute node functionality|
|performance-overlay|performance tuning|
|debug-overlay|temporary debugging|

Avoid creating overly complex overlays that mix unrelated responsibilities.

#### 8.4.3 Use Overlay Priority Carefully

Overlay priority determines which configuration takes precedence when conflicts occur.

Recommended priority structure:

|Overlay|Priority|Purpose|
|---|---|---|
|base-overlay|100|baseline configuration|
|compute-overlay|200|node functionality|
|performance-overlay|400|tuning|
|debug-overlay|800|temporary overrides|
|emergency-overlay|1000|urgent fixes|

Higher priority overlays should be used cautiously to avoid unintended overrides.

#### 8.4.4 Use Overlays for Environment Differences

Instead of modifying roles for different environments, use overlays.

Examples:

|Environment|Overlay|
|---|---|
|production|compute-overlay|
|testing|compute-overlay + debug-overlay|
|performance testing|compute-overlay + performance-overlay|

This approach keeps roles reusable and reduces duplication.

#### 8.4.5 Avoid Assigning Roles Directly to Nodes

Roles should be applied through overlays rather than directly to nodes.

Benefits:

- easier management
- better consistency
- clearer configuration structure

Direct role assignment should be reserved for exceptional cases.

### 8.5 Role and Overlay Interaction

Roles and overlays work together to produce the final node configuration.

The process follows a layered approach:

```text
Software Image
       ↓
Node Category
       ↓
Roles
       ↓
Overlays (priority order)
       ↓
Node overrides
```

Important points:

- overlays assign roles to nodes
- nodes receive all roles from all applied overlays
- conflicts are resolved using overlay priority
- final configuration is determined through hierarchical evaluation

### 8.6 Example Design for a GPU Cluster

A clean design for a GPU cluster might include:

#### Roles

```text
base-os
time-sync
monitoring-agent
slurm-config
slurm-client
slurm-controller
gpu-driver
cuda-runtime
nfs-client
dcgm-agent
rdma-stack
nccl-tuning
```

#### Overlays

|Overlay|Roles|
|---|---|
|base-overlay|base-os, time-sync, monitoring-agent|
|slurm-overlay|slurm-config, munge|
|compute-overlay|slurm-client, gpu-driver, cuda-runtime, nfs-client, dcgm-agent|
|performance-overlay|rdma-stack, nccl-tuning|
|login-overlay|nfs-client, container-runtime|
|head-overlay|slurm-controller|

This design is:

- modular
- scalable
- easy to maintain

### 8.7 Common Anti-Patterns

Avoid the following design mistakes:

#### Multiple Roles Managing the Same File

Leads to configuration conflicts and instability.

#### Role Explosion

Too many roles create unnecessary complexity.

#### Hardcoding Configuration in Roles

Prevents reuse and flexibility.

#### Overloading Overlays

Mixing unrelated roles in a single overlay reduces clarity.

#### Ignoring Priority Rules

Incorrect overlay priority can override critical configurations.

### 8.8 Operational Benefits

Following these best practices provides several advantages:

- improved configuration consistency
- easier troubleshooting
- reduced risk of outages
- simplified cluster scaling
- better collaboration among administrators

A clean design enables the cluster to grow without increasing operational complexity.

### 8.9 Summary

Designing roles and overlays correctly is essential for effective configuration management in BCM.

Key recommendations include:

- create modular and reusable roles
- avoid role explosion through parameterization
- enforce single ownership of configuration files
- use overlays to group and manage roles
- apply overlay priority carefully
- maintain a clear and consistent configuration hierarchy

By following these best practices, administrators can build scalable and maintainable HPC clusters that are easier to operate and troubleshoot.

---

## 9. Example: Managing a 128-GPU HPC Cluster with BCM

### 9.1 Introduction

This chapter presents a practical example of how NVIDIA Base Command Manager (BCM) can be used to design and operate a **128-GPU HPC cluster**. The goal is to demonstrate how concepts such as **categories, roles, overlays, provisioning, and operations** come together in a real-world deployment.

This example reflects a typical AI training cluster used for distributed deep learning workloads.

### 9.2 Cluster Architecture Overview

Assume the cluster contains **128 GPUs**, organized as follows:

|Node Type|Count|Description|
|---|---|---|
|Head node|1|BCM controller + Slurm controller|
|Login nodes|2|user access|
|Compute nodes|16|each with 8 GPUs (16 × 8 = 128 GPUs)|
|Storage nodes|2|shared storage (NFS/Lustre)|
|Monitoring node|1|metrics and observability|

Total nodes: **22**

### 9.3 Network Design

The cluster uses multiple networks:

|Network|Purpose|
|---|---|
|Management network|provisioning, SSH, BCM communication|
|High-speed network|GPU communication (InfiniBand / RDMA)|
|Storage network|data access|

Compute nodes are connected using a **leaf-spine topology** to minimize latency and maximize bandwidth.

### 9.4 Node Categories

Each node belongs to a category that defines its baseline configuration.

|Category|Purpose|
|---|---|
|head-node|cluster control|
|login-node|user access|
|compute-gpu|GPU compute nodes|
|storage-node|shared storage|
|monitor-node|monitoring services|

Example mapping:

```text
head01 → head-node
login01 → login-node
login02 → login-node
compute01–compute16 → compute-gpu
storage01–02 → storage-node
monitor01 → monitor-node
```

### 9.5 Software Images

Software images provide minimal OS baselines.

|Image|Used By|
|---|---|
|rocky8-base|login, monitor|
|gpu-baseos|compute nodes|
|storage-base|storage nodes|

Images contain:

- operating system
- basic networking
- minimal utilities

All additional functionality is applied via roles.

### 9.6 Roles Design

Roles are designed as modular capabilities.

#### Core Roles

```text
base-os
time-sync
logging-agent
monitoring-agent
```

#### Slurm Roles

```text
munge
slurm-config
slurm-client
slurm-controller
```

#### GPU Roles

```text
gpu-driver
cuda-runtime
dcgm-agent
```

#### Storage Roles

```text
nfs-server
nfs-client
```

(or Lustre roles in larger deployments)

#### Container Roles

```text
container-runtime
nvidia-container-runtime
```

#### Performance Roles

```text
rdma-stack
nccl-tuning
```

### 9.7 Overlay Design

Overlays group roles and define node functionality.

#### Base Overlay (Priority 100)

Applied to all nodes:

```text
base-os
time-sync
logging-agent
monitoring-agent
```

#### Slurm Overlay (Priority 200)

Applied to all nodes:

```text
munge
slurm-config
```

#### Compute Overlay (Priority 300)

Applied to compute-gpu:

```text
slurm-client
gpu-driver
cuda-runtime
nfs-client
dcgm-agent
container-runtime
nvidia-container-runtime
```

#### Head Overlay (Priority 300)

Applied to head-node:

```text
slurm-controller
nfs-client
```

#### Login Overlay (Priority 300)

Applied to login-node:

```text
nfs-client
container-runtime
```

#### Storage Overlay (Priority 300)

Applied to storage-node:

```text
nfs-server
```

#### Performance Overlay (Priority 500)

Applied to compute-gpu:

```text
rdma-stack
nccl-tuning
```

This ensures high-performance communication for distributed training.

### 9.8 Provisioning Workflow

When a new compute node is added:

1. node boots via PXE
2. BCM assigns IP via DHCP
3. node downloads bootloader via TFTP
4. provisioning environment starts
5. OS installed from gpu-baseos
6. node reboots
7. BCM applies roles and overlays
8. services start

After provisioning, the node appears in Slurm:

```bash
sinfo
```

Output:

```text
compute01 idle
compute02 idle
...
compute16 idle
```

### 9.9 Workload Execution

User workflow:

1. user logs into login node:

```bash
ssh login01
```

2. submit job:

```bash
sbatch train.sh
```

3. Slurm schedules job on compute nodes
4. nodes execute distributed training

Compute nodes already have:

- GPU drivers
- CUDA
- NCCL tuning
- shared storage access

This ensures jobs run without manual setup.

### 9.10 Monitoring and Observability

Monitoring stack includes:

- Prometheus
- Grafana
- NVIDIA DCGM

Metrics collected:

- GPU utilization
- CPU usage
- memory usage
- network throughput
- job statistics

Administrators use dashboards to:

- detect performance bottlenecks
- identify failing nodes
- track cluster utilization

### 9.11 Maintenance Workflow

Example: upgrading GPU driver.

1. update gpu-driver role
2. test on small subset of nodes
3. drain nodes:

```bash
scontrol update NodeName=compute01 State=DRAIN
```

4. apply update
5. reboot node
6. validate:

```bash
nvidia-smi
```

7. resume node:

```bash
scontrol update NodeName=compute01 State=RESUME
```

8. repeat for remaining nodes

This rolling update avoids cluster downtime.

### 9.12 Scaling the Cluster

If the cluster expands:

- add new compute nodes
- assign category `compute-gpu`
- apply existing overlays

No redesign is required.

BCM automatically ensures:

- consistent configuration
- correct software versions
- proper integration with Slurm

### 9.13 Common Operational Scenarios

#### Node Failure

- node marked DOWN in Slurm
- remove from scheduling
- diagnose hardware/software
- reprovision if necessary

#### Performance Degradation

- check GPU utilization
- verify network performance
- analyze storage throughput

#### Configuration Changes

- update role or overlay
- apply changes
- validate cluster state

### 9.14 Benefits of This Design

This cluster design provides:

- **consistency** across all nodes
- **scalability** for future growth
- **modularity** through roles and overlays
- **efficient operations** using automation
- **high performance** for distributed workloads

### 9.15 Summary

This example demonstrates how BCM can be used to manage a 128-GPU HPC cluster using structured configuration management.

Key elements include:

- clearly defined node categories
- modular roles for capabilities
- overlays for functional grouping
- automated provisioning workflow
- integration with Slurm and monitoring systems

By applying these design patterns, administrators can build and operate large-scale HPC clusters that are reliable, scalable, and easy to manage.

---

## 10. How DevOps / SRE Engineers Use BCM

### 10.1 Introduction

In HPC and AI infrastructure, DevOps and Site Reliability Engineering (SRE) practices play a critical role in ensuring that large-scale clusters operate reliably, efficiently, and with minimal manual intervention.

NVIDIA Base Command Manager (BCM) provides the foundation for managing cluster infrastructure, while DevOps and SRE engineers build automation, reliability practices, and operational workflows on top of it.

This chapter explains how DevOps and SRE engineers use BCM in real-world environments, including daily operations, automation strategies, incident response, and system design.

### 10.2 Role of DevOps and SRE in HPC

In traditional cloud environments, DevOps focuses on application delivery, while SRE focuses on reliability. In HPC environments, these responsibilities expand to include:

- infrastructure provisioning at scale
- performance optimization
- workload reliability
- hardware lifecycle management
- cluster-wide configuration consistency

Unlike cloud systems, HPC clusters are often **bare-metal environments**, which require deeper interaction with hardware, networking, and system-level configuration.

### 10.3 Infrastructure as Code with BCM

DevOps engineers treat BCM configuration as **Infrastructure as Code (IaC)**.

Key elements include:

- roles
- overlays
- categories
- configuration templates

These components are:

- version-controlled (e.g., Git)
- reviewed through pull requests
- tested before deployment

Example workflow:

```text
Modify role → Commit to Git → Code review → Test cluster → Deploy to production
```

This approach ensures:

- traceability of changes
- reproducibility of infrastructure
- safer updates

### 10.4 Automation and CI/CD

DevOps engineers build CI/CD pipelines around BCM configuration.

Typical pipeline stages:

1. validate configuration changes
2. test role deployment on staging nodes
3. run automated checks (e.g., service health)
4. promote changes to production

Example automation tasks:

- updating GPU drivers
- deploying new Slurm configurations
- applying security patches
- rolling out performance tuning

Automation reduces manual errors and speeds up operations.

### 10.5 Cluster Provisioning and Scaling

DevOps engineers use BCM to automate cluster provisioning.

Tasks include:

- defining node categories
- assigning roles and overlays
- provisioning new nodes via PXE
- scaling compute capacity

When new hardware is added:

1. register node in BCM
2. assign category
3. trigger provisioning
4. node becomes available automatically

This enables rapid scaling of HPC clusters.

### 10.6 Observability and Monitoring

SRE engineers focus on **observability**, which includes metrics, logs, and alerts.

BCM integrates with monitoring tools to collect data such as:

- node health
- GPU utilization
- job success rates
- network latency
- storage performance

Common tools:

- Prometheus
- Grafana
- NVIDIA DCGM

SRE engineers build dashboards and alerts to detect:

- node failures
- performance degradation
- abnormal resource usage

This enables proactive system management.

### 10.7 Incident Response

Handling production incidents is a core responsibility of SRE engineers.

Common incidents include:

- nodes failing in scheduler
- GPU errors
- network issues
- storage bottlenecks

SRE engineers follow structured workflows:

1. identify affected nodes
2. isolate the issue (scheduler, node, network, storage)
3. collect logs and metrics
4. apply fixes
5. restore service

Example:

```bash
scontrol update NodeName=compute05 State=DRAIN
```

This prevents additional failures while debugging.

### 10.8 Reliability Engineering

SRE engineers define reliability practices such as:

- Service Level Objectives (SLOs)
- failure recovery procedures
- redundancy strategies
- automated health checks

In HPC environments, reliability often focuses on:

- job success rate
- cluster availability
- GPU utilization efficiency

BCM supports these practices by providing consistent configuration and centralized control.

### 10.9 Performance Optimization

HPC clusters require continuous performance tuning.

DevOps and SRE engineers optimize:

- CPU and GPU utilization
- network performance (RDMA tuning)
- storage throughput
- job scheduling efficiency

Examples of tuning activities:

- adjusting NCCL parameters
- optimizing RDMA settings
- tuning kernel parameters
- improving data locality

These optimizations directly impact workload performance.

### 10.10 Change Management

Managing changes safely is critical in production clusters.

SRE engineers use controlled processes:

- staging environments for testing
- canary deployments (small subset of nodes)
- rolling updates
- rollback strategies

Example workflow:

```text
Test change → Apply to 2 nodes → Validate → Roll out gradually → Monitor → Complete deployment
```

This reduces the risk of cluster-wide failures.

### 10.11 Security and Access Control

DevOps engineers also manage security aspects of the cluster.

Key tasks include:

- controlling SSH access
- managing user authentication
- securing container environments
- applying security patches

BCM helps enforce consistent security policies across nodes.

### 10.12 Collaboration with Engineers and Researchers

HPC DevOps/SRE engineers work closely with:

- data scientists
- researchers
- machine learning engineers

They support users by:

- providing stable compute environments
- optimizing job performance
- troubleshooting job failures

Effective communication is essential to align infrastructure capabilities with user needs.

### 10.13 Differences from Cloud DevOps

HPC DevOps differs from cloud-based DevOps in several ways:

|Aspect|HPC|Cloud|
|---|---|---|
|Infrastructure|bare metal|virtualized|
|scaling|fixed capacity|elastic|
|networking|RDMA / InfiniBand|TCP/IP|
|performance focus|latency and throughput|scalability|
|provisioning|PXE-based|API-based|

DevOps engineers transitioning to HPC must adapt to these differences.

### 10.14 Daily Workflow of an HPC SRE

A typical day may include:

- checking cluster health dashboards
- responding to alerts
- troubleshooting failed jobs
- updating configuration roles
- provisioning new nodes
- performing maintenance tasks

Most work focuses on maintaining stability and improving performance.

### 10.15 Summary

DevOps and SRE engineers use BCM as a foundation for managing HPC infrastructure. By applying principles such as automation, observability, and reliability engineering, they ensure that clusters operate efficiently and consistently.

Key responsibilities include:

- infrastructure as code and automation
- monitoring and observability
- incident response and troubleshooting
- performance optimization
- safe change management

By combining BCM capabilities with DevOps and SRE practices, organizations can operate large-scale HPC and AI clusters with high reliability and efficiency.
