# Module 07 — Author a PolicyGenerator policy

## Brief Overview

This module moves from consuming policies to authoring them. Participants scaffold a new operator policy (Web Terminal) using the generator script, read the 4 files it creates, render the PolicyGenerator directory locally the same way the CMP sidecar does, and run the integration test to validate the label contract. The integration test has 5 stages and catches undeclared labels before they reach a cluster.

## Audience and Time

- **Personas:** Platform engineers, SREs, policy authors
- **Prerequisites:** Module 6 completed (troubleshooting understood), Go 1.23+ installed
- **Duration:** 30 minutes

## Learning Objectives

- Scaffold a new operator policy with `generate-operator-policy.sh` instead of writing the 4 files by hand
- Render a PolicyGenerator directory locally using kustomize with the PolicyGenerator plugin, matching the CMP sidecar behavior
- Explain what the integration test's 5 stages enforce (Helm render, hub template resolution, spoke template resolution, resolved-YAML validation, label contract) and why an undeclared label fails CI

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Scaffold the policy (generator script) | 10 min |
| 2 | Read the 4 files and render locally | 10 min |
| 3 | Run the integration test and fix the label contract | 10 min |

## Detailed Steps

1. Look up the Web Terminal operator package in OperatorHub: `oc get packagemanifest web-terminal`
2. Run `./scripts/generate-operator-policy.sh web-terminal web-terminal --channel fast --namespace openshift-operators`
3. List the generated directory: `policies/stable/web-terminal/` — confirm kustomization.yaml, policy-generator-config.yaml, placement.yaml, manifests/operator-install/kustomization.yaml
4. Read `kustomization.yaml` — always 3 lines, declares the PolicyGenerator config as a generator
5. Read `policy-generator-config.yaml` — defines the policy graph, references `${...}` placeholders for CMP substitution
6. Read `placement.yaml` — matchExpressions selects clusters with `autoshift.io/web-terminal: 'true'`
7. Read `manifests/operator-install/kustomization.yaml` — nested kustomization calling the shared `components/operator-install` Helm chart with hub template expressions
8. Install the PolicyGenerator toolchain: `make install-policy-generator`
9. Stage a copy with placeholder substitution, then render with kustomize: confirm output contains Policy, Placement, PlacementBinding
10. Run the integration test: `go test -C tools -tags integration ./internal/resolver/...` — expect failure at label contract stage (6 missing keys)
11. Add all 6 web-terminal labels to `autoshift/values/clustersets/_example.yaml`
12. Re-run the integration test — all 5 stages pass

## Key Takeaways

- A PolicyGenerator directory has 4 key files: kustomization.yaml, policy-generator-config.yaml, placement.yaml, and the nested manifests/ kustomization calling the shared chart
- Local rendering requires placeholder substitution (what the CMP sidecar does) before kustomize can process `evaluationInterval` as a valid Go duration
- The integration test has 5 stages: Helm render → hub template resolution → spoke template resolution → YAML validation → label contract
- Every `autoshift.io/<key>` a policy consumes must be declared in `_example.yaml` — this is the label contract, enforced in CI
- `tools/` is its own Go module — use `go test -C tools` to run from the repo root

## Infrastructure Notes

- Go 1.23+ is required for the integration test
- The Web Terminal operator is used as the example because it is small and not already in the repository
- The generated policy only exists locally — auto-discovery by the ApplicationSet requires a commit and push to the upstream repo
