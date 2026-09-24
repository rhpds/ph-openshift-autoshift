# Module 10 — Your spoke cluster joined the fleet

## Brief Overview

The spoke cluster declared in Module 3 has been building in the background for ~6 modules. This module collects the result: the cluster registered itself as a ManagedCluster, the cluster-labels policy stamped it from the managed clusterset, Placements selected it, and every enabled policy was enforced on a machine the participant never logged into. Then the cluster is torn down — this step is mandatory to release AWS resources.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** Module 3 completed (spoke cluster declared), Module 9 completed (fleet operations understood)
- **Duration:** 15 minutes

## Learning Objectives

- Confirm a Hive-provisioned cluster registered as a ManagedCluster and inherited its clusterset's labels automatically via the cluster-labels policy
- Explain how a cluster that was never manually configured ended up running the operators declared in the managed clusterset values file
- Deprovision the cluster using the 2-step process (set createCluster: 'false', then delete ClusterDeployment) and confirm AWS resources are released

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Watch the cluster join the fleet | 5 min |
| 2 | Teardown (mandatory) | 10 min |

## Detailed Steps

1. Check the ClusterDeployment status: `oc get clusterdeployment spoke-aws-1 -n spoke-aws-1`
2. Watch the ManagedCluster appear and reach `JOINED: True`: `oc get managedcluster spoke-aws-1 -w`
3. Confirm the clusterset label was stamped: expect `cluster.open-cluster-management.io/clusterset=managed`
4. Inspect the full set of `autoshift.io/` labels on spoke-aws-1 — they match the managed clusterset values file
5. List policies replicated to the spoke-aws-1 namespace: `oc get policy -n spoke-aws-1` — confirm NFD and other enabled policies are Compliant
6. Understand the chain: values file → ConfigMap → label → Placement → policy — all automatic, no manual intervention
7. **Teardown Step 1:** Set `createCluster: 'false'` in spoke-aws-1.yaml and apply — this stops the policy from recreating the ClusterDeployment
8. Wait for the rendered-config ConfigMap to show `createCluster: "false"`
9. **Teardown Step 2:** Delete the ClusterDeployment: `oc delete clusterdeployment spoke-aws-1 -n spoke-aws-1` — Hive runs an uninstall job
10. Watch the deprovision (~10-15 minutes) — EC2 instances, VPC, NAT gateway, Elastic IPs, and Route 53 records are removed
11. **Teardown Step 3:** Detach and clean up: `oc delete managedcluster spoke-aws-1` and `oc delete namespace spoke-aws-1`
12. Confirm the namespace is gone and only local-cluster remains in `oc get managedcluster`

## Key Takeaways

- A cluster declared in a values file inherits its entire configuration from the clusterset it joins — the label flow from Module 2 runs automatically on registration
- `createCluster: 'false'` does NOT destroy anything — it only stops the policy from emitting the ClusterDeployment. Deletion requires explicitly deleting the ClusterDeployment after the policy stops managing it
- Teardown is a 2-step process: disable the policy first, then delete the resource. Doing either alone leaves a running cluster
- FIPS mode forbids ed25519 SSH keys — the install fails at bootstrap with a non-obvious error message

## Infrastructure Notes

- Teardown is mandatory — leaving the spoke running incurs ongoing AWS billing
- The ClusterDeployment deletion triggers Hive's uninstall job, which removes all AWS resources (VPC, EC2, NAT, EIP, Route 53)
- The ManagedCluster and namespace must be deleted separately — Hive does not clean up hub-side objects
