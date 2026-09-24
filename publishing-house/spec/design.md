# Getting to know AutoShift

## Overview

Hands-on workshop teaching platform engineers to govern OpenShift clusters at scale using Red Hat Advanced Cluster Management (ACM), OpenShift GitOps (Argo CD), and the AutoShift IaC framework. Participants bootstrap a hub cluster, deploy AutoShift's Helm chart, and work through 10 modules that progress from initial setup to fleet-wide operations including spoke cluster provisioning on AWS. The lab demonstrates AutoShift's label-driven policy model: a values file feeds ConfigMaps, ConfigMaps feed ManagedCluster labels, labels drive Placements, and Placements dispatch policies — all without modifying policy code.

## Target Audience

- **Role:** Platform engineers, SREs, and infrastructure operators managing OpenShift fleets
- **Experience level:** Intermediate
- **What they already know:** OpenShift cluster administration (cluster-admin level), Helm 3.x charting, Git workflows, basic Kubernetes resource model
- **What they don't know:** ACM governance and policy framework, AutoShift's label-driven configuration model, PolicyGenerator authoring, OCI chart publishing for disconnected environments, fleet-wide operator lifecycle management

## Prerequisites

- OpenShift cluster-admin experience (navigating oc CLI, reading resource YAML)
- Helm 3.x (templating, values files, install/upgrade)
- Git, make, curl, jq, yq installed locally
- Go 1.23+ (required for Module 7 integration test only)
- AWS account with permissions to create VPCs, EC2 instances, Route 53 records, and IAM roles (for spoke cluster provisioning)
- Can the lab validate these automatically? Partially — the showroom environment provides oc, helm, and CLI tools; AWS credentials and Go are checked at the point of use

## Learning Objectives

1. Bootstrap Red Hat ACM, OpenShift GitOps, and AutoShift on a hub cluster and explain the three-phase deployment model
2. Trace the label flow from a values file through a ConfigMap to a ManagedCluster label and explain how Placements use those labels to dispatch policies
3. Enable, pin, and disable operators fleet-wide using the 5-label operator contract without modifying any policy code
4. Author a new PolicyGenerator policy using the generator script, render it locally with kustomize, and pass the integration test's 5-stage pipeline
5. Publish AutoShift to an OCI registry and redeploy from it, and flip the fleet to mirrored catalog sources for disconnected environments

## Content Type

Lab (hands-on)

## Products & Technologies

- Red Hat OpenShift Container Platform 4.22
- Red Hat Advanced Cluster Management for Kubernetes
- Red Hat OpenShift GitOps (Argo CD)
- Red Hat OpenShift Data Foundation (ODF MCG standalone / NooBaa)
- Red Hat Quay
- Operator Lifecycle Manager (OLM) / OperatorPolicy
- PolicyGenerator (ACM governance)
- Helm 3.x
- Kustomize
- Hive (cluster provisioning)
- AWS (EC2, VPC, Route 53, IAM)

## Module Map

| Module | Title | Duration |
|--------|-------|----------|
| 1 | Bootstrap AutoShift | 30 min |
| 2 | Trace the label flow | 20 min |
| 3 | Start the long-running builds | 20 min |
| 4 | Enable a feature with labels alone | 30 min |
| 5 | Config blocks and per-cluster overrides | 20 min |
| 6 | Operate and troubleshoot | 30 min |
| 7 | Author a PolicyGenerator policy | 30 min |
| 8 | OCI publishing and disconnected environments | 30 min |
| 9 | Fleet operations capstone | 20 min |
| 10 | Your spoke cluster joined the fleet | 15 min |
| — | **Total hands-on** | **4 hours 5 min** |
| — | Intro / overview / details | ~15 min |
| — | **Total lab** | **~4 hours 20 min** |

## Difficulty Level

Intermediate

## Environment

**Learner view:** A single OpenShift 4.22 cluster with cluster-admin access. No operators are pre-installed — participants install Red Hat ACM, OpenShift GitOps, and AutoShift themselves in Module 1 as part of the bootstrap exercise. The AutoShift Helm chart repository is cloned locally. AWS credentials are pre-staged in a Secret for spoke cluster provisioning in later modules.

**Automation needed:** Yes

- Bare OpenShift 4.22 cluster provisioned with cluster-admin access (no additional operators pre-installed)
- AutoShift Helm chart repository cloned to student workspace
- AWS credentials Secret pre-created in cluster-install-secrets namespace
- Pull secret available for spoke cluster provisioning
- CLI tools available: oc, helm, git, make, curl, jq, yq, Go

## Infrastructure Requirements

- **Cloud provider:** AWS (EC2)
- **Cluster type:** Compact (schedulable control plane, no dedicated workers)
- **OCP version:** 4.22
- **Topology:** Per-student
- **Sizing:** 3x m5a.8xlarge compact (32 vCPU, 128 GB RAM each) — hub cluster with schedulable control plane, no dedicated worker nodes. Additionally, the lab provisions a SNO spoke cluster on AWS (Module 3) which requires additional AWS resources (1 EC2 instance, VPC, NAT gateway, Elastic IP, Route 53 zone).
- **Automation approach:** Combo (Helm + ArgoCD for AutoShift deployment, ACM policies for cluster configuration)
- **AI/MaaS:** None
- **External services:** github.com (AutoShift source repo), quay.io (OCI chart registry)
- **Non-GA products:** None (all products are GA)
- **Open environment credentials:** Required — AWS credentials with available Elastic IPs for spoke cluster provisioning. Credentials must not be reused from prior sessions to avoid Elastic IP exhaustion.
- **FIPS:** Preferred (toggle available) — not mandatory

## Assessment Strategy

Each module includes verify steps with specific `oc` commands that confirm the expected state. The showroom provides solve and validate buttons per exercise. Validation checks include:
- Module 1: ArgoCD Application Synced + Healthy, ApplicationSet creating per-policy Applications
- Module 2: ManagedCluster labels match values file, PlacementDecision selects expected clusters
- Module 3: Secrets created in correct namespaces, ClusterDeployment and QuayRegistry CRs exist
- Module 4: OperatorPolicy Compliant, Subscription channel matches expected value
- Module 5: rendered-config ConfigMap contains override values
- Module 6: Policy diagnostic chain passes all 5 layers, dry run mode toggles remediation action
- Module 7: Integration test passes all 5 stages (render, hub resolution, spoke resolution, YAML validation, label contract)
- Module 8: Rendered Helm charts exist in .helm-charts/policies/, OCI push succeeds
- Module 9: ManagedClusterSet membership verified, upgrade policy labels confirmed
- Module 10: ManagedCluster JOINED=True with correct clusterset label, teardown confirmed (namespace gone)
