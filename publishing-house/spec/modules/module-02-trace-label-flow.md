# Module 02 — Trace the label flow

## Brief Overview

This module traces the complete path a configuration value takes from a YAML file to a running cluster. Participants observe the null state (no clusterset assignment), then assign local-cluster to a clusterset and watch as the cluster-labels policy stamps every label from the clusterset values onto the ManagedCluster. The module ends with a deletion exercise proving that `mustonlyhave` enforcement propagates label removals.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** Module 1 completed (AutoShift deployed on hub)
- **Duration:** 20 minutes

## Learning Objectives

- Observe the null state where no clusterset is assigned and confirm no autoshift.io/ labels exist on the ManagedCluster
- Assign a clusterset label to local-cluster and trace the chain: values file → ConfigMap → ManagedCluster labels → Placement → PlacementDecision
- Prove that removing a label from the values file causes `mustonlyhave` enforcement to delete it from the ManagedCluster

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Observe null state (no clusterset assignment) | 5 min |
| 2 | Assign clusterset and trace the chain | 10 min |
| 3 | Prove deletion propagates | 5 min |

## Detailed Steps

1. Check ManagedCluster `local-cluster` for any `autoshift.io/` labels — expect none
2. Check the `cluster.open-cluster-management.io/clusterset` label — expect `default`
3. Read the `cluster-set.hub` ConfigMap in the policy namespace — it was created by the AutoShift chart from the values file
4. Apply `oc label managedcluster local-cluster cluster.open-cluster-management.io/clusterset=hub --overwrite`
5. Watch the cluster-labels policy read the ConfigMap and stamp all labels from the hub clusterset onto local-cluster
6. Verify `autoshift.io/` labels appear on the ManagedCluster using `oc get managedcluster local-cluster -o yaml | grep autoshift.io/`
7. Confirm Placements that select the `hub` clusterset now include local-cluster in their PlacementDecision
8. Add a test label to `hub-minimal.yaml`, re-run `helm upgrade`, and confirm it appears on the ManagedCluster
9. Remove the test label from `hub-minimal.yaml`, re-run `helm upgrade`
10. Confirm the label is deleted from the ManagedCluster — `mustonlyhave` enforcement removed it

## Key Takeaways

- The label flow has 4 steps: values file → ConfigMap → ManagedCluster label → Placement match
- `mustonlyhave` means the ConfigMap is the source of truth — any label not in it is actively removed
- Clusterset membership is what activates policies on a cluster; without it, no policies apply
- The chain is fully declarative — changing the values file and re-applying is sufficient

## Infrastructure Notes

- Uses local-cluster (the hub itself) as the target — no additional clusters needed for this module
