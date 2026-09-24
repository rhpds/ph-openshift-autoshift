# Module 09 — Fleet operations capstone

## Brief Overview

This module closes the lab by showing that 4 apparently distinct fleet operations — gradual rollout, declarative cluster assignment, OpenShift cluster upgrades, and hub-of-hubs topology — all reduce to the same primitive: a cluster moves from one clusterset to another. Most of this module is conceptual (single hub cluster), with targeted exercises on the lab cluster to verify clusterset membership and upgrade policy state.

## Audience and Time

- **Personas:** Platform engineers, SREs, fleet operators
- **Prerequisites:** Module 8 completed (OCI and disconnected understood)
- **Duration:** 20 minutes

## Learning Objectives

- Explain how gradual rollout, declarative cluster assignment, OpenShift cluster upgrades, and hub-of-hubs topology all reduce to clusterset membership
- Describe how to roll a change out gradually using versioned clustersets, and what reverting a commit does and does not undo (labels reverse, but operator upgrades and provisioned clusters do not)
- State the single ACM constraint (a cluster is managed by exactly 1 ACM) that explains hub-of-hubs topology

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Gradual rollout with versioned clustersets | 5 min |
| 2 | Declarative cluster assignment and OpenShift upgrades | 10 min |
| 3 | Hub-of-hubs topology and values repo separation | 5 min |

## Detailed Steps

1. Review the current ManagedClusterSets on the lab cluster: `oc get managedclusterset` — note `hub`, `managed`, `default`, `global`
2. Explain versioned clustersets: `versionedClusterSets: true` appends a version suffix (e.g., `hub-0-0-1`), enabling 2 releases to run side by side with no shared clustersets
3. Confirm local-cluster's clusterset membership label
4. Explain declarative cluster assignment via `config.clusterSet` and `config.versionTag` in per-cluster values files — owner-guarded to prevent 2 releases fighting over a cluster
5. Explain the 4 upgrade policies (channel → allowed → upgrade → status) and their dependency chain
6. Verify the upgrade policies exist in the policy namespace but have blank compliance (no cluster opted in)
7. Confirm local-cluster does not carry the `autoshift.io/openshift-upgrade` label
8. Explain the single ACM constraint: a cluster is managed by exactly 1 ACM → a spoke hub cannot manage itself → the hub above must deploy its configuration
9. Explain cross-namespace lookups (cluster-wide lookup by label, not namespace-scoped) for hub-of-hubs policies
10. Emphasize the values repo separation: policy code in `autoshiftv2`, site configuration in a separate repo, Argo CD stitches them with multi-source Application

## Key Takeaways

- Every fleet operation reduces to clusterset membership — moving a cluster between clustersets is the universal lever
- Reverting a commit reverses declared intent (clusterset membership, policy application) but does NOT reverse already-applied work (operator upgrades stay, provisioned clusters stay)
- The single ACM constraint (1 cluster : 1 ACM) determines the entire hub-of-hubs topology
- Production deployments separate policy code (autoshiftv2 repo) from site configuration (values repo) — upgrades become a `targetRevision` bump

## Infrastructure Notes

- This module is primarily conceptual — the lab runs a single hub, so multi-cluster operations are explained rather than executed
- The upgrade policies are deployed but have no opted-in clusters, so compliance columns are blank by design
