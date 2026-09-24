# Module 04 — Enable a feature with labels alone

## Brief Overview

This module demonstrates the core AutoShift operator contract: 5 labels control the full lifecycle of any operator in the fleet. Participants enable Node Feature Discovery (NFD) using labels, then learn to control operator versions (allow-list vs pin, label mode vs config mode), and finally disable NFD by flipping a single label. The key insight is that no policy code changes — only values file labels.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** Module 2 completed (label flow understood)
- **Duration:** 30 minutes

## Learning Objectives

- Enable NFD using the 5-label operator contract: enable flag, subscription-name, channel, source, source-namespace
- Control operator versions using the allow-list (versions label) vs pin approaches, and distinguish label mode from config mode
- Disable an operator by setting its enable label to `'false'` and explain what remains on the cluster (Subscription and namespace persist)

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Enable NFD with the 5-label contract | 10 min |
| 2 | Control versions — allow-list, pin, label vs config mode | 10 min |
| 3 | Disable NFD | 10 min |

## Detailed Steps

1. Add 5 NFD labels to `hub-minimal.yaml`: `node-feature-discovery: 'true'`, `node-feature-discovery-subscription-name: nfd`, `node-feature-discovery-channel: stable`, `node-feature-discovery-source: redhat-operators`, `node-feature-discovery-source-namespace: openshift-marketplace`
2. Apply and wait for the NFD OperatorPolicy to reach Compliant
3. Confirm the NFD Subscription exists with the correct channel
4. Confirm the NFD instance CR is deployed and NFD pods are running
5. Add `node-feature-discovery-version` label to set a version allow-list — explain that this constrains which CSVs OLM is permitted to install
6. Explain the difference between Subscription `Manual` approval (label mode) and OperatorPolicy `Automatic` approval — OperatorPolicy re-approves on reconcile, so Manual in the Subscription is effectively automatic
7. Remove the version label — OLM picks the latest CSV on the channel
8. Set `node-feature-discovery: 'false'` in the values file
9. Apply and confirm the OperatorPolicy is removed from the cluster namespace
10. Check that the Subscription and namespace remain — disabling removes the policy, not the installed resources

## Key Takeaways

- The 5-label contract (enable, subscription-name, channel, source, source-namespace) is the universal interface for every operator in AutoShift
- Version control via labels uses an allow-list — setting a version doesn't pin; it constrains which CSVs are accepted
- Disabling an operator sets `enable: 'false'`, which removes the policy from the cluster's Placement. The Subscription and namespace remain — cleanup requires manual removal or a separate cleanup policy
- No policy code was modified in any step — all control is through labels in the values file

## Infrastructure Notes

- NFD is used as the example operator because it is lightweight and ships in redhat-operators
- The version label value must be a valid CSV name (e.g., `nfd.4.22.0-202501010000`)
