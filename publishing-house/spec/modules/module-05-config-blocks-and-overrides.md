# Module 05 — Config blocks and per-cluster overrides

## Brief Overview

Modules 2-4 used labels for scalar OLM fields. This module introduces config blocks — the second configuration form — for nested, structured data that cannot be expressed as a single label value. Participants add a User Workload Monitoring (UWM) config block, read the rendered-config ConfigMap, and then apply a per-cluster override to prove the precedence chain: cluster beats clusterset beats defaults.

## Audience and Time

- **Personas:** Platform engineers, SREs
- **Prerequisites:** Module 4 completed (label-based operator management understood)
- **Duration:** 20 minutes

## Learning Objectives

- Explain why 2 configuration forms exist: labels for scalar OLM fields, config blocks for nested structured data
- Add a UWM config block to a clusterset values file and read the result in the rendered-config ConfigMap
- Apply a per-cluster override and confirm the precedence chain: cluster values beat clusterset values beat defaults

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Why 2 config forms exist | 5 min |
| 2 | Add UWM config block and read rendered-config | 10 min |
| 3 | Per-cluster override (precedence chain) | 5 min |

## Detailed Steps

1. Explain that labels handle scalar OLM fields (channel, source, etc.) but cannot represent nested YAML structures like monitoring retention settings or storage class configurations
2. Add a `config.userWorkloadMonitoring` block to `hub-minimal.yaml` with retention and resource limit settings
3. Apply with `helm upgrade --install`
4. Read the `local-cluster.rendered-config` ConfigMap in the policy namespace — confirm the UWM config block appears in the rendered output
5. Create or edit `autoshift/values/clusters/local-cluster.yaml` with a `config.userWorkloadMonitoring` override that changes the retention value
6. Apply with all values files including the per-cluster file
7. Read the rendered-config ConfigMap again — confirm the per-cluster value overrides the clusterset value
8. Remove the per-cluster override file entry and re-apply
9. Confirm the rendered-config reverts to the clusterset-level value

## Key Takeaways

- Labels handle scalar values (OLM fields); config blocks handle nested structured data (monitoring config, storage classes, GitOps settings)
- The rendered-config ConfigMap is the merged result of all config blocks for a cluster — it is what policies actually read
- Precedence chain: cluster-level config beats clusterset-level config beats global defaults
- Config blocks follow the same values file pattern as labels — change the file, re-apply, done

## Infrastructure Notes

- No additional infrastructure needed — uses the hub cluster (local-cluster)
- The rendered-config ConfigMap is created by the AutoShift chart for every cluster that has config blocks
