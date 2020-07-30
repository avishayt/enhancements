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
assisted_installer_cluster_creations{platform="metal", version="4.6"} 50
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `version` - represents OCP minor releases

##### assisted_installer_cluster_installation_started
`assisted_installer_cluster_installation_started` represents the clusters that started the actual installation process. It signifies that a user was able to fulfill the requirements and boot the hosts, but not necessarily that the installation process completed in any way.

```
# HELP assisted_installer_cluster_installation_started represents the number of clusters whose installation began.
# TYPE assisted_installer_cluster_installation_started counter
assisted_installer_cluster_installation_started{platform="metal", version="4.6"} 40
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `version` - represents OCP minor releases
 
##### assisted_installer_cluster_installation_minutes
 `assisted_installer_cluster_installation_minutes` represents an invocation of the installation, from start to finish.

```
# HELP assisted_installer_cluster_installation_minutes represents an invocation of the installation, from start to finish.
# TYPE assisted_installer_cluster_installation_minutes histogram
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="1"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="5"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="10"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="15"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="20"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="25"} 0
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="30"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="35"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="40"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="45"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="50"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="55"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="60"} 1
assisted_installer_cluster_installation_minutes_bucket{platform="metal", result="success", version="4.6", agentversion="1.0", le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that a user successfully installed a 4.6 cluster, which took between 30 and 35 minutes to complete installation. 

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents OCP minor releases
 * `agentversion` - represents minor versions of the assisted installer agent
 * `le` - the standard Prometheus label for histogram buckets, this represents how long the installation took to complete, from 1 minute (or less) to 60 minutes (or more), in increments of 5 minutes.

##### assisted_installer_host_installation_phase_minutes
 `assisted_installer_host_installation_stage_minutes` represents a stage of the installation process for a host.

```
# HELP assisted_installer_host_installation_stage_minutes represents a stage of the installation process for a host.
# TYPE assisted_installer_host_installation_stage_minutes histogram
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker", result="success", version="4.6", agentversion="1.0", le="1"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="5"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="10"} 0
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="15"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="20"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="25"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="30"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="35"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="40"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="45"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="50"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="55"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="60"} 1
assisted_installer_host_installation_stage_minutes_bucket{platform="metal", stage="Rebooting", role="worker",result="success", version="4.6", agentversion="1.0", le="+Inf"} 1
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
 * `version` - represents OCP minor releases
 * `agentversion` - represents minor versions of the assisted installer agent
 * `le` - the standard Prometheus label for histogram buckets, this represents how long the installation took to complete, from 1 minute (or less) to 60 minutes (or more), in increments of 5 minutes.

##### assisted_installer_cluster_dns_types
`assisted_installer_cluster_dns_types` represents the number of clusters installed with private vs hosted DNS.

```
# HELP assisted_installer_cluster_dns_types represents the number of clusters installed with private vs hosted DNS.
# TYPE assisted_installer_cluster_dns_types counter
assisted_installer_cluster_creations{platform="metal", result="success", version="4.6", hosted="true"} 50
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases
 * 'hosted' - DNS is hosted ("true") or not ("false")

##### assisted_installer_cluster_hosts
`assisted_installer_cluster_hosts` represents the number of hosts that took part in an installation.  Dividing by the number of installations gives the average number of hosts.

```
# HELP assisted_installer_cluster_hosts represents the number of hosts in an installation.
# TYPE assisted_installer_cluster_hosts counter
assisted_installer_cluster_hosts{platform="metal", role="master", result="success", version="4.6"} 3
```

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases

##### assisted_installer_cluster_host_cores
 `assisted_installer_cluster_host_cores` represents the number of CPU cores for an installed host.

```
# HELP assisted_installer_cluster_host_cores represents the number of CPU cores for an installed host.
# TYPE assisted_installer_cluster_host_cores histogram
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="1"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="2"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="4"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="8"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="16"} 0
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="32"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="64"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="128"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="256"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="512"} 1
assisted_installer_cluster_host_cores_bucket{platform="metal", role="master",result="success",version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, had between 32 and 64 CPU cores.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents how many CPU cores there were, from 1 to 512 (or more) in powers of 2.

##### assisted_installer_cluster_host_ram_gib
 `assisted_installer_cluster_host_ram_gib` represents the amount of RAM in GiB for an installed host.

```
# HELP assisted_installer_cluster_host_ram_gib represents the amount of RAM in GiB for an installed host.
# TYPE assisted_installer_cluster_host_ram_gib histogram
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="8"} 0
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="16"} 0
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="32"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="64"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="128"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="256"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="512"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="1024"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="2048"} 1
assisted_installer_cluster_host_ram_gib_bucket{platform="metal", role="master",result="success",version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, had between 32 and 64 GiB of RAM.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents how much RAM there was in GiB, from 8 (the minimum) to 2 TiB (or more) in powers of 2.

##### assisted_installer_cluster_host_disk_gb
 `assisted_installer_cluster_host_disk_gb` represents the size of a disk in GB in an installed host.

```
# HELP assisted_installer_cluster_host_disk_gb represents the size of a disk in GB in an installed host.
# TYPE assisted_installer_cluster_host_disk_gb histogram
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="250"} 0
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="500"} 0
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="1000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="2000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="4000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="8000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="16000"} 1
assisted_installer_cluster_host_disk_gb_bucket{platform="metal", role="master",result="success",version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, has a disk with capacity between 1TB and 2TB.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents the capacity of a disk in GB, from 250 (or less) to 16TB (or more), exponentially.

##### assisted_installer_cluster_host_nic_gbps
 `assisted_installer_cluster_host_nic_gbps` represents the bandwidth of a NIC in Gbps in an installed host.

```
# HELP assisted_installer_cluster_host_nic_gbps represents the size of a NIC in Gbps in an installed host.
# TYPE assisted_installer_cluster_host_nic_gbps histogram
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master",result="success",version="4.6",le="1"} 0
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master",result="success",version="4.6",le="10"} 0
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master",result="success",version="4.6",le="40"} 1
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master",result="success",version="4.6",le="100"} 1
assisted_installer_cluster_host_nic_gbps_bucket{platform="metal", role="master",result="success",version="4.6",le="+Inf"} 1
```
The example above represents a single sample. This sample indicates that this host, that was successfully-installed with version 4.6 and a master role, has a NIC with bandwidth between 40 and 100 Gbps.

###### Label Values
 * `platform` - the platform used for the installation (metal, vmware, aws, etc.)
 * `role` - the role assigned to the host for the installation, could be one of:
    - master
    - worker
 * `result` - as a first stage will be binary (success/failure), once we support error codes we would use those
 * `version` - represents minor releases
  * `le` - the standard Prometheus label for histogram buckets, this represents the bandwidth of a NIC in Gbps, from 1 to 100, covering the most common bandwidths.

### Risks and Mitigations

What are the risks of this proposal and how do we mitigate. Think broadly. For
example, consider both security and how this will impact the larger OKD
ecosystem.

How will security be reviewed and by whom? How will UX be reviewed and by whom?

Consider including folks that also work outside your immediate sub-project.

## Design Details

### Open Questions [optional]

This is where to call out areas of the design that require closure before deciding
to implement the design.  For instance, 
 > 1. This requires exposing previously private resources which contain sensitive
  information.  Can we do this? 

### Test Plan

**Note:** *Section not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy. Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations).

### Graduation Criteria

**Note:** *Section not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. Initial proposal
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this
enhancement:
- Maturity levels - `Dev Preview`, `Tech Preview`, `GA`
- Deprecation

Clearly define what graduation means.

#### Examples

These are generalized examples to consider, in addition to the aforementioned
[maturity levels][maturity-levels].

##### Dev Preview -> Tech Preview

- Ability to utilize the enhancement end to end
- End user documentation, relative API stability
- Sufficient test coverage
- Gather feedback from users rather than just developers

##### Tech Preview -> GA 

- More testing (upgrade, downgrade, scale)
- Sufficient time for feedback
- Available by default

**For non-optional features moving to GA, the graduation criteria must include
end to end tests.**

##### Removing a deprecated feature

- Announce deprecation and support policy of the existing feature
- Deprecate the feature

### Upgrade / Downgrade Strategy

If applicable, how will the component be upgraded and downgraded? Make sure this
is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to keep previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to make use of the enhancement?

### Version Skew Strategy

How will the component handle version skew with other components?
What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- During an upgrade, we will always have skew among components, how will this impact your work?
- Does this enhancement involve coordinating behavior in the control plane and
  in the kubelet? How does an n-2 kubelet without this feature available behave
  when this feature is used?
- Will any other components on the node change? For example, changes to CSI, CRI
  or CNI may require updating that component before the kubelet.

## Implementation History

Major milestones in the life cycle of a proposal should be tracked in `Implementation
History`.

## Drawbacks

The idea is to find the best form of an argument why this enhancement should _not_ be implemented.

## Alternatives

Similar to the `Drawbacks` section the `Alternatives` section is used to
highlight and record other possible approaches to delivering the value proposed
by an enhancement.

## Infrastructure Needed [optional]

Use this section if you need things from the project. Examples include a new
subproject, repos requested, github details, and/or testing infrastructure.

Listing these here allows the community to get the process for these resources
started right away.
