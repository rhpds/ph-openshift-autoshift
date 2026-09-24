# Module 08 — OCI publishing and disconnected environments

## Brief Overview

This module covers what happens to PolicyGenerator directories at release time. They are pre-rendered into stock Helm charts, packaged as OCI artifacts, and pushed to a registry. In OCI mode, Argo CD pulls these charts without needing a CMP sidecar at all — which removes the ACM-before-GitOps ordering constraint. The module also covers flipping an entire fleet to mirrored catalog sources with 2 labels, and generating an ImageSetConfiguration for `oc mirror`.

## Audience and Time

- **Personas:** Platform engineers, SREs, release engineers
- **Prerequisites:** Module 7 completed (PolicyGenerator authoring understood)
- **Duration:** 30 minutes

## Learning Objectives

- Render every PolicyGenerator directory into stock Helm charts with `make render-policy-charts` and explain why OCI mode needs no CMP sidecar
- Publish AutoShift to an OCI registry (or dry-run the push) and observe the ApplicationSet switch from git generator to list generator
- Flip an entire fleet to mirrored catalog sources with 2 labels (`disconnected-mirror` and `mirror-catalog-suffix`) and explain how the mirror list is derived from the shared ternary hub template expression

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Render PolicyGenerator directories into Helm charts | 10 min |
| 2 | (Optional) Install registry, publish, and redeploy from OCI | 10 min |
| 3 | Flip the fleet to mirrored catalog sources | 10 min |

## Detailed Steps

1. Install the PolicyGenerator toolchain: `make install-policy-generator`
2. Render all PG directories: `make render-policy-charts VERSION=0.0.0-dev`
3. List the output under `.helm-charts/policies/` — each PG directory became a packaged `.tgz` Helm chart
4. Compare a source PG directory (e.g., `policies/stable/advanced-cluster-management/`) with its rendered chart — note the chart has `Chart.yaml`, `values.yaml`, `templates/policies.yaml`, and `files/rendered.yaml`
5. Dry-run push: `make push-charts DRY_RUN=true VERSION=0.0.0-dev` — see the 3 registry paths (bootstrap, main, policies)
6. (Optional full path) Install ODF MCG + Quay on the hub, create a Quay superuser, render with a real version, push to the local Quay registry
7. (Optional) Switch AutoShift to OCI mode by setting `autoshiftOciRepo` and `autoshiftOciVersion`
8. (Optional) Inspect the ApplicationSet to confirm it switched from `git:` generator to `list:` generator
9. Add `disconnected-mirror: 'true'` and `mirror-catalog-suffix: 'mirror'` labels to the clusterset
10. Observe all OperatorPolicy `source:` fields now end in `-mirror` — the shared ternary expression rewired them without any policy code change
11. Generate an ImageSetConfiguration: `make generate-imageset` — the script reads `-subscription-name` labels to discover which operators to mirror
12. Revert disconnected labels before the next module to restore policy compliance

## Key Takeaways

- OCI mode pre-renders PG directories at release time — Argo CD pulls stock Helm charts, no CMP sidecar needed, no ACM-before-GitOps constraint
- The ApplicationSet switches generators between modes: `git:` files generator for source installs, `list:` generator reading `policy-list.txt` for OCI
- 2 labels (`disconnected-mirror` + `mirror-catalog-suffix`) rewrite every operator policy's catalog source through a shared ternary hub template — no policy code changes
- `generate-imageset-config.sh` reads `-subscription-name` labels to build the mirror manifest — this is why the subscription-name label is mandatory

## Infrastructure Notes

- The full OCI path (Part 2) requires ~40 minutes and real cluster headroom for ODF MCG + Quay
- 3 fallback paths are available: local render only, dry-run push, or inspect the published release at `oci://quay.io/autoshift/`
- Quay creates repositories as private by default — either make them public or register OCI credentials with ArgoCD
