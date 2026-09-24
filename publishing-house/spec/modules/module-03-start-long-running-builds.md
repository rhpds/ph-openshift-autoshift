# Module 03 — Start the long-running builds

## Brief Overview

This module declares 3 long-running resources in parallel: an ODF MCG standalone registry (NooBaa-only, no Ceph), a Red Hat Quay registry backed by that object storage, and an AWS spoke cluster (SNO, FIPS-enabled). All 3 take significant time to provision, so they are started early and run in the background while subsequent modules cover other topics. The module teaches the pattern of declaring intent now and collecting results later.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** Module 2 completed (label flow understood, local-cluster in hub clusterset)
- **Duration:** 20 minutes

## Learning Objectives

- Declare an ODF MCG standalone registry by adding ODF and Quay operator labels to the clusterset values file
- Build the cluster-install source secret (SSH ECDSA key, AWS credentials, pull secret) and explain the FIPS constraint on key algorithms
- Declare a spoke cluster by writing a per-cluster values file with the cluster-install config block

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Declare the registry (ODF MCG + Quay labels) | 5 min |
| 2 | Build the cluster-install source secret | 5 min |
| 3 | Declare the spoke cluster (SNO on AWS) | 10 min |

## Detailed Steps

1. Add ODF and Quay operator labels to `hub-minimal.yaml` under the hub clusterset — enable flags, subscription names, channels, sources
2. Add the `odf-multi-cloud-gateway: standalone` label to activate MCG-only mode (no Ceph)
3. Apply with `helm upgrade --install` and confirm ODF and Quay policies appear in the policy namespace
4. Generate an ECDSA SSH keypair (`ssh-keygen -t ecdsa -b 521`) — ed25519 is forbidden in FIPS mode
5. Create a Kubernetes Secret in `cluster-install-secrets` namespace containing: AWS access key, AWS secret key, SSH private key, SSH public key, and pull secret
6. Write `autoshift/values/clusters/spoke-aws-1.yaml` with the cluster-install config block: cluster name, base domain, region, instance type, FIPS enabled, credential references
7. Apply with all 4 values files (`-f global.yaml -f hub-minimal.yaml -f managed-minimal.yaml -f spoke-aws-1.yaml`)
8. Confirm the ClusterDeployment CR is created in the spoke-aws-1 namespace
9. Note the provision pod starting — the install runs in the background (~40 minutes)
10. Verify the QuayRegistry and NooBaa resources are being created in their respective namespaces

## Key Takeaways

- Long-running operations (cluster installs, operator deployments) are started early and run concurrently — declare intent now, collect results later
- ODF MCG standalone brings only NooBaa's object storage — no Ceph, no dedicated storage nodes
- FIPS mode constrains SSH key algorithm to ECDSA or RSA; the install fails at bootstrap if ed25519 is used
- The cluster-install source secret uses a single-secret shape with 4 fields (credential, SSH private, SSH public, pull secret)

## Infrastructure Notes

- AWS credentials must have permissions for VPC, EC2, ELB, Route 53, S3, and IAM role creation
- The spoke cluster is SNO (single-node OpenShift) on AWS — 1 EC2 instance with combined control plane and worker
- Elastic IPs are allocated for the spoke — they must be released during teardown in Module 10
