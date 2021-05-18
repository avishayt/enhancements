---
title: agent-late-binding
authors:
  - "@avishayt"
reviewers:
  - "@mhrivnak"
  - "@filanov"
  - "@ronniel1"
approvers:
  - TBD
creation-date: 2021-05-19
last-updated: 2021-05-19
status: implementable
see-also:
  - "enhancements/installer/connected-assisted-installer.md"
---

# Agent Late Binding

## Release Signoff Checklist

- [ ] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

In the current implementation of Assisted Installer, each Discovery ISO is
associated with an existing cluster resource (a cluster definition that may not
yet be an installed OCP cluster).  When an agent comes up, it registers with
the Assisted Service and is automatically associated with that same cluster
resource.  The only way to associate that host with a different cluster is to
boot it with an ISO associated with the desired cluster.  We can see that the
process of configuring the infrastructure is closely tied to cluster creation.

The late binding feature splits the process into two parts, each potentially
performed by a different persona.  The Discovery ISO is created by an
infrastructure admin, who then boots the hosts.  Each host becomes visible to
the cluster creators but not associated with any specific cluster resource.  At
a later time, a cluster creator can create or expand a cluster using available
hosts.

## Motivation

The benefits of late binding are as follows:
- Adjusts the flow to suit the two different personas (infrastructure
  administrator and cloud creator).
- Allows for a more cloud-like use case where compute resources of different
  flavors can be consumed, similar to IBM Satellite.
- Allows agent-based installation to fit more naturally into a MAPI or CAPI
  model.  While the details of such an integration are not in the scope of
  this enhancement, the idea is that hosts are booted, thus creating a
  collection of unassigned agents, and the machine creation involves choosing one
  such agent and beginning its installation.
- Reduces the number of Discovery ISOs, as there is no need to generate and
  download one for each cluster installation.  Each ISO incurs capacity
  overhead for the service, and management overhead for the user which conflicts
  with our ease-of-use goal.
- Allows the user to see their inventory and then make decisions on how to
  build clusters based on hardware and connectivity.  At a later stage, the
  service can make recommendations.  This would be a layer of logic above late
  binding and is out of scope for this enhancement.
- Enables the user story where the Discovery ISO is loaded at the factory, and
  then discovered at a later time.

### Goals

- Make the InfraEnv and Agent resources independent from the ClusterDeployment.
  A single InfraEnv should have the potential to boot any number of hosts,
  whose Agents register with no relation to any particular ClusterDeployment.  An
  Agent may be assigned to a ClusterDeployment at a later time.
- An Agent may be associated with a different ClusterDeployment at any time.

### Non-Goals

- Once a host boots from disk, booting the Discovery ISO again to return it to
  the collection of free Agents is considered to be a manual step.  There are
  several options that we will explore whereby this can be done automatically,
  but that discussion is out of scope for this enhancement.
- At a later stage, Agents may be automatically assigned to clusters based on
  their discovered inventory and/or network configurations. This is future work
  and not in the scope of this enhancement.

## Proposal

The first major change is related to the Discovery ISO.  The only API change
(which is non-breaking) is that ClusterRef will no longer mandatory.  Once an
agent registers with the service, its corresponding Agent CR must be created in
the namespace of the InfraEnv which generated its Discovery ISO.  The
controller currently relies on the fact that an agent is always associated with
a cluster, and an InfraEnv is always associated with a cluster, to associate
Agents with InfraEnvs.  Now that these associations are optional, the
InfraEnv's identity must be encoded in the ISO, and the agent must pass it to
the service upon registration.  When an agent registers, the backend will
create the Agent CR in the InfraEnv's namespace, and if the InfraEnv references
a ClusterDeployment, the Agent will automatically be associated with that
ClusterDeployment.

The second major change is to the Agent CRD, where the ClusterRef is no longer
mandatory.  This association may now be deleted or changed dynamically, unless
the Agent is part of an ongoing cluster installation, in which case the
installation must first be cancelled.  If the host had started/completed its
installation, a new Agent Condition would indicate that the user needs to boot
the host into the Discovery ISO once again.

Agent validations that do not depend on being part of a cluster will run for
Agents that are both associated with clusters and those that are not.  Examples
of these include minimum hardware for any role, synced NTP, and registry
access.  Validations that do depend on being part of a cluster, such as
connectivity checks, will run only for Agents associated with clusters.

### User Stories

#### Story 1

As an Infrastructure Admin, I want to create a Discovery ISO and use it to boot
a pool of hosts for use by Cluster Creators to install OpenShift on.

#### Story 2

As an Infrastructure Admin, I want to add Discovery ISOs to hosts as a boot
option, and then ship the hosts to remote locations where they will be booted
at a later time for use by Cluster Creators to install OpenShift on.

#### Story 3

As a Cluster Creator, I want to view a collection of available Agents and use
them for OpenShift cluster creation or expansion.

#### Story 4

As a Cluster Creator, I want to reassign an Agent from one OpenShift cluster to
another.

### Implementation Details/Notes/Constraints [optional]

- It would be beneficial to move the discovery image management to a separate
  service at some point, both to allow it to be scaled independently and to
  reduce the very large scope of the existing service.  The late binding work
  should be done with this separation in mind, even if the actual separation is
  an orthogonal task.

- While the Agent and ClusterDeployment resources will be viewed as independent
  to API users, to avoid a large DB and code migration, internally Agents will
  still always be associated with clusters in the SQL DB.

- The REST API will be overhauled as well to separate the image and host
  resources from the cluster resource.

### Risks and Mitigations

## Design Details

### Open Questions [optional]

### Test Plan

**Note:** *Section not required until targeted at a release.*

### Graduation Criteria

**Note:** *Section not required until targeted at a release.*

### Upgrade / Downgrade Strategy

Existing CRs should continue to work without modification.

### Version Skew Strategy

## Implementation History

## Drawbacks

## Alternatives

## Infrastructure Needed [optional]

