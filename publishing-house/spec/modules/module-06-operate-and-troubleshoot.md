# Module 06 — Operate and troubleshoot

## Brief Overview

Every change so far has worked. Real fleets break. This module provides a repeatable 5-layer diagnostic sequence for tracing policy failures from the Argo CD Application down to the ConfigurationPolicy or OperatorPolicy root cause. Participants also learn the dry run safety switch (which changes 2 things simultaneously), policy dependencies as structural safety, and deliberate breakage to understand silent drift from missing labels with `| default` expressions.

## Audience and Time

- **Personas:** Platform engineers, SREs, operations staff
- **Prerequisites:** Module 5 completed (config blocks and overrides understood)
- **Duration:** 30 minutes

## Learning Objectives

- Walk the 5-layer diagnostic chain from Argo CD Application → Policy → Placement/PlacementDecision → replicated policy → ConfigurationPolicy/OperatorPolicy to isolate the root cause of a NonCompliant policy
- Enable and disable dry run mode and explain the 2 things it changes simultaneously: REMEDIATION=inform on all PolicyGenerator policies, and prune=false + selfHeal=false on Argo CD sync policy
- Explain what a policy `dependencies` block does and why it prevents a config policy from applying before its operator is installed

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | The diagnostic toolkit (5-layer chain) | 10 min |
| 2 | Dry run mode as the fleet-wide safety switch | 10 min |
| 3 | Deliberate breakage — missing label with default expression | 10 min |

## Detailed Steps

1. Walk through the diagnostic chain on a healthy policy (policy-selfmanagedhub-labels): check Argo CD Application sync status, check Policy compliance on the hub, check PlacementDecision cluster selection, check replicated policy in local-cluster namespace, check inner ConfigurationPolicy status
2. Learn 2 fixes for stuck policies: force re-evaluation via trigger annotation, and delete stuck replicated policy to force re-creation
3. Optionally enable the AutoShift console plugin by adding the `autoshift-console: 'true'` label
4. Set `dryRun: true` in global.yaml and apply — observe all PolicyGenerator policies flip from enforce to inform
5. Monitor the inform count climbing over ~6 minutes as the ApplicationSet re-renders
6. Revert dryRun to false and confirm policies return to enforce
7. Re-enable NFD (set enable label back to 'true')
8. Remove the `node-feature-discovery-channel` label from hub-minimal.yaml and apply
9. Diagnose using the toolkit — discover the policy is still Compliant because the hub template has `| default "stable"`
10. Understand the risk: silent drift, not failure — the channel is a fallback, not the intended explicit value
11. Restore the channel label to make intent explicit
12. Read the `dependencies` block on `policy-nfd-instance-deploy` — it waits for `policy-nfd-operator-install` to be Compliant before dispatching

## Key Takeaways

- The diagnostic chain runs top to bottom: Application → Policy → Placement → replicated policy → inner policy. The error is usually 1 layer above the actual cause
- Dry run changes 2 things simultaneously: remediation mode to inform (report only) and Argo CD sync to no-prune/no-selfHeal
- A missing label does not always cause failure — `| default` expressions silently substitute a fallback. The risk is silent drift, not a visible error
- Policy dependencies prevent config policies from applying before their operator is installed — Pending is not a failure, it is the policy waiting

## Infrastructure Notes

- Dry run propagation takes ~6 minutes on a hub of this size — expect the inform count to climb gradually
- The AutoShift console plugin is optional and community-supported; all exercises remain CLI-only
