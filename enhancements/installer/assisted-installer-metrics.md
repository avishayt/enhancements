---
title: assisted-installer-telemetry-metrics
authors:
  - "@atraeger"
reviewers:
  - TBD
  - "@alicedoe"
approvers:
  - TBD
  - "@oscardoe"
creation-date: 2020-07-22
last-updated: 2020-07-22
status: implementable
see-also:
  - "/enhancements/installer/connected-assisted-installer.md"
---

# Assisted Installer Telemetry Metrics

## Release Signoff Checklist

- [X] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

The assisted installer, [previously proposed and accepted](connected-assisted-installer.md), collects informative installation metrics. This enhancement proposes that the assisted installer would push metrics related to service usage, user workflows, and installation statistics. This enhancement focuses on the design of the metrics to be pushed to Prometheus.

## Motivation

Metrics from the assisted installer service would help engineers and product managers to understand different user workflows, environments, experiences, difficulties, and feature usage. This would benefit not only future versions of the assisted installer but also the OpenShift installer itself.

### Goals

1. Describe a set of metrics to fulfill the user stories described below, while keeping cardinality reasonable.
2. Describe how the metrics will be pushed to Prometheus.

### Non-Goals

1. Collected metrics will not allow analysis of individual installations.

## Proposal

### User Stories

#### Telemetry Analyst

As a telemetry analyst I would like to create queries which show why users failed to install OpenShift, whether due to difficulties in fulfilling pre-requisites, UX issues, or bugs.

As a telemetry analyst I would like to create queries which break down the installation time experienced by users.

As a telemetry analyst, I would like to understand what phases and configurations cause the majority of the installation failures.

As a telemetry analyst I would like to create queries which show useful characteristics about the cluster hardware, to understand how it may affect installation success rates, and to focus further OpenShift development.

### Implementation Details/Notes/Constraints [optional]

The installer does not fit a typical Prometheus use case in that:
* metrics will be pushed rather than scraped

#### Metrics definitions
##### assisted_installer_cluster_creations
`assisted_installer_cluster_creations` represents the creation of a cluster resource in the assisted installer service. It signifies that a user wished to begin an installation, but not necessarily that the installation process was started or completed (perhaps the user was unable to fulfill requirements or get hosts booted).

```
# HELP assisted_installer_cluster_creations represents the number of cluster resources created in the assisted installer service.
# TYPE assisted_installer_cluster_creations counter
assisted_installer_cluster_creations{platform="metal", ocp_version="4.6"} 50
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `ocp_version` - represents OCP minor releases

##### assisted_installer_cluster_installation_started
`assisted_installer_cluster_installation_started` represents the clusters that started the actual installation process. It signifies that a user was able to fulfill the requirements and boot the hosts, but not necessarily that the installation process completed in any way.

```
# HELP assisted_installer_cluster_installation_started represents the number of clusters whose installation began.
# TYPE assisted_installer_cluster_installation_started counter
assisted_installer_cluster_installation_started{platform="metal", ocp_version="4.6"} 40
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `ocp_version` - represents OCP minor releases

##### assisted_installer_cluster_installation_minutes
 `assisted_installer_cluster_installation_minutes` represents an invocation of the installation, from start to finish.

```
# HELP assisted_installer_cluster_installation_minutes represents an invocation of the installation, from start to finish.
# TYPE assisted_installer_cluster_installation_minutes histogram
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="1"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="5"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="10"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="15"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="20"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="25"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="30"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="35"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="40"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="45"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="50"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="55"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="60"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", ocp_version="4.6", agent_version="1.0", le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that a user successfully installed a 4.6 cluster, which took between 30 and 35 minutes to complete installation. 

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents OCP minor releases
 * `agent_version` - represents minor versions of the assisted installer agent
 * `le` - the standard Prometheus label for histogram buckets, this represents how long the installation took to complete, from 1 minute (or less) to 60 minutes (or more), in increments of 5 minutes.

##### assisted_installer_host_installation_phase_minutes
 `assisted_installer_host_installation_stage_minutes` represents a stage of the installation process for a host.

```
# HELP assisted_installer_host_installation_stage_minutes represents a stage of the installation process for a host.
# TYPE assisted_installer_host_installation_stage_minutes histogram
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="1"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="5"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="10"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="15"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="20"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="25"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="30"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="35"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="40"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="45"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="50"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="55"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="60"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", ocp_version="4.6", agent_version="1.0", le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that during the installation of a 4.6 cluster, the phase where this worker host rebooted took between 10 and 15 minutes to complete.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `stage` - a non-final stage in the installation, could be one of:
    - Starting installation
    - Waiting for control plane
    - Installing
    - Writing image to disk
    - Rebooting
    - Waiting for ignition
    - Configuring
    - Joined
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
    - bootstrap-master
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents OCP minor releases
 * `agent_version` - represents minor versions of the assisted installer agent
 * `le` - the standard Prometheus label for histogram buckets, this represents how long the installation took to complete, from 1 minute (or less) to 60 minutes (or more), in increments of 5 minutes.

##### assisted_installer_cluster_dns_types
`assisted_installer_cluster_dns_types` represents the number of clusters installed with private vs hosted DNS.

```
# HELP assisted_installer_cluster_dns_types represents the number of clusters installed with private vs hosted DNS.
# TYPE assisted_installer_cluster_dns_types counter
assisted_installer_cluster_creations{platform="metal", result="success", ocp_version="4.6", hosted="true"} 50
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases
 * 'hosted' - DNS is hosted ("true") or not ("false")

##### assisted_installer_cluster_hosts
`assisted_installer_cluster_hosts` represents the number of hosts that took part in an installation.  Dividing by the number of installations gives the average number of hosts.

```
# HELP assisted_installer_cluster_hosts represents the number of hosts in an installation.
# TYPE assisted_installer_cluster_hosts counter
assisted_installer_cluster_hosts{platform="metal", role="master", result="success", ocp_version="4.6"} 3
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases

##### assisted_installer_cluster_host_cores
 `assisted_installer_cluster_host_cores` represents the number of CPU cores for an installed host.

```
# HELP assisted_installer_cluster_host_cores represents the number of CPU cores for an installed host.
# TYPE assisted_installer_cluster_host_cores histogram
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="1"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="2"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="4"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="8"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="16"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="32"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="64"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="128"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="256"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="512"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, had between 32 and 64 CPU cores.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents how many CPU cores there were, from 1 to 512 (or more) in powers of 2.

##### assisted_installer_cluster_host_ram_gib
 `assisted_installer_cluster_host_ram_gib` represents the amount of RAM in GiB for an installed host.

```
# HELP assisted_installer_cluster_host_ram_gib represents the amount of RAM in GiB for an installed host.
# TYPE assisted_installer_cluster_host_ram_gib histogram
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="8"} 0
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="16"} 0
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="32"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="64"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="128"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="256"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="512"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="1024"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="2048"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, had between 32 and 64 GiB of RAM.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents how much RAM there was in GiB, from 8 (the minimum) to 2 TiB (or more) in powers of 2.

##### assisted_installer_cluster_host_disk_gb
 `assisted_installer_cluster_host_disk_gb` represents the size of a disk in GB in an installed host.

```
# HELP assisted_installer_cluster_host_disk_gb represents the size of a disk in GB in an installed host.
# TYPE assisted_installer_cluster_host_disk_gb histogram
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="250"} 0
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="500"} 0
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="1000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="2000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="4000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="8000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="16000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, has a disk with capacity between 1TB and 2TB.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents the capacity of a disk in GB, from 250 (or less) to 16TB (or more), exponentially.

##### assisted_installer_cluster_host_nic_gbps
 `assisted_installer_cluster_host_nic_gbps` represents the bandwidth of a NIC in Gbps in an installed host.

```
# HELP assisted_installer_cluster_host_nic_gbps represents the size of a NIC in Gbps in an installed host.
# TYPE assisted_installer_cluster_host_nic_gbps histogram
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="1"} 0
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="10"} 0
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="40"} 1
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="100"} 1
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master", result="success", ocp_version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, has a NIC with bandwidth between 40 and 100 Gbps.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `ocp_version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents the bandwidth of a NIC in Gbps, from 1 to 100, covering the most common bandwidths.

### Risks and Mitigations

 > 1. Security: no information that can identify a particular installation/user/organization will be collected.
 > 2. Cardinality: keep the amount of label values minimal.

## Design Details

### Open Questions

 > 1. We are considering adding the service version to each metric, but are wondering if the cardinality will be too high given that we plan to release every 2 weeks or so.

### Test Plan

TBD

### Graduation Criteria

TBD

### Upgrade / Downgrade Strategy

TBD

### Version Skew Strategy

TBD

## Implementation History

TBD

## Drawbacks

TBD

## Alternatives

TBD

## Infrastructure Needed [optional]

TBD
