# Module 01 — Bootstrap AutoShift

## Brief Overview

This module walks participants through the three-phase deployment that stands up AutoShift on a hub cluster. Phase 1 bootstraps ACM (order matters — ACM before GitOps so the CMP sidecar can resolve). Phase 2 bootstraps GitOps. Phase 3 deploys the AutoShift Helm chart itself. By the end, participants have a running AutoShift instance with an ApplicationSet that auto-discovers policy directories and creates one Argo CD Application per policy.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** OpenShift cluster-admin access, Helm 3.x basics, oc CLI familiarity
- **Duration:** 30 minutes

## Learning Objectives

- Clone the AutoShift repository and inspect the values file structure (global, clusterset, per-cluster)
- Bootstrap ACM and then GitOps in the correct order, and explain why ACM must come first (CMP sidecar dependency)
- Deploy the AutoShift Helm chart and observe the ApplicationSet fan-out creating per-policy Argo CD Applications

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Clone repo and inspect values files | 10 min |
| 2 | Bootstrap ACM then GitOps | 10 min |
| 3 | Deploy AutoShift and watch ApplicationSet fan-out | 10 min |

## Detailed Steps

1. Clone the autoshiftv2 repository to the local workspace
2. Inspect `autoshift/values/global.yaml` — note the `dryRun`, `autoshiftGitRepo`, and `autoshiftOciVersion` settings
3. Inspect the reference clusterset values files under `autoshift/values/clustersets/`
4. Run the ACM bootstrap Helm install — creates the ACM operator Subscription and MultiClusterHub CR
5. Wait for MultiClusterHub to reach Running status
6. Run the GitOps bootstrap Helm install — creates the GitOps operator Subscription and ArgoCD CR
7. Wait for the ArgoCD instance to become available
8. Deploy AutoShift with `helm upgrade --install` using global.yaml and a hub-minimal clusterset values file
9. Observe the ApplicationSet `autoshift-policies` created in the `openshift-gitops` namespace
10. List the Argo CD Applications — each corresponds to a directory under `policies/`
11. Verify at least one Application shows `Synced` and `Healthy`

## Key Takeaways

- AutoShift deploys in 3 phases: ACM → GitOps → AutoShift chart. The order is not arbitrary — ACM must exist before GitOps because the CMP sidecar needs ACM's deployment to resolve
- The ApplicationSet uses a git file generator that auto-discovers `policies/*/kustomization.yaml` directories
- Values are layered: global → clusterset → per-cluster. Each `-f` flag in the `helm upgrade` command adds a layer
- No policies are applied to any cluster yet — the Placements have no matching clusters until labels are stamped

## Infrastructure Notes

- The hub cluster needs ACM and GitOps operators available in the default OperatorHub catalog sources
- FIPS mode is enabled on the hub — SSH keys must be ECDSA or RSA (no ed25519)
