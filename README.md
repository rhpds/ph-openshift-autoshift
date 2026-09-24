# ph-openshift-autoshift

Hands-on workshop teaching platform engineers to govern OpenShift clusters at scale using Red Hat ACM, OpenShift GitOps (Argo CD), and the AutoShift IaC framework. 10 modules (~4 hours) progress from hub bootstrap through fleet operations.

Participants learn to manage fleet-wide configuration through a label-driven policy model: enabling operators with labels alone, authoring PolicyGenerator policies, publishing to OCI registries (including disconnected environments), and provisioning AWS spoke clusters that inherit configuration automatically.

Key topics: ACM + GitOps bootstrap, label-driven operator management, per-cluster config overrides, custom PolicyGenerator authoring with Kustomize, OCI publishing via Red Hat Quay, gradual rollout strategies, hub-of-hubs patterns, and spoke cluster lifecycle.

AWS is required — the lab provisions a real AWS spoke cluster to demonstrate multi-cluster fleet management, and the hub runs ODF (NooBaa MCG) on AWS storage.

Technologies: OpenShift, ACM, Argo CD, Helm, OLM/OperatorPolicy, PolicyGenerator, Quay, ODF, AWS.

Target: Platform engineers and SREs. Intermediate pace — assumes comfort with OpenShift, Helm, and Git.

**Owner:** bcarr-rh
**Migrated from:** https://github.com/auto-shift/autoshift-showroom

---

## What was set up

1. Repository created (migrated from existing Showroom repo)
2. `catalog-info.yaml` added to repository
3. Registered in Developer Hub catalog
4. Orchestrator workflow started — your AI-guided content pipeline is running!

## What happens next

Claude will walk you through the entire content lifecycle — from intake and spec creation, through Jira tracking and reviews, all the way to a published lab on RHDP. Just follow the prompts!

## Getting started

### DevSpaces (recommended)

1. Open in DevSpaces: `https://devspaces.apps.ocpv-infra02.wdc07.infra.demo.redhat.com#https://github.com/rhpds/ph-openshift-autoshift`
2. Use Claude via the **extension** or the **CLI**:
   - **Extension:** Click the **Claude** icon in the sidebar, click **New Session**. If the Claude icon is not visible, open **Extensions** (`Ctrl/Cmd+Shift+X`), find **Claude Code for VS Code** under the DevSpaces section, click it, then click **Enable (Workspace)**.
   - **CLI:** Open a terminal and run `claude`
3. Run `/rhdp-publishing-house` — and you're off!

### Local machine

1. Install the skills:
   ```
   git clone -b prod https://github.com/rhpds/rhdp-publishing-house-skills.git ~/.claude/skills/publishing-house
   ```
2. Clone the repo:
   ```
   git clone https://github.com/rhpds/ph-openshift-autoshift
   ```
3. `cd ph-openshift-autoshift`
4. Start Claude CLI: `claude`
5. Run `/rhdp-publishing-house` — and you're off!
