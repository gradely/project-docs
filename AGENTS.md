# Documentation contribution guidance

## Purpose

This repository records cross-repository facts, integration contracts, critical flows, and reviewable operational gaps. It is not a replacement for service-local runbooks or implementation documentation.

## Verification standard

- Cite the owning repository, branch, and source path for every material technical claim.
- Label historical material and environment-dependent assertions clearly.
- Do not infer deployment topology, public URLs, gateway rules, secrets, or database state from source code alone.
- Do not copy credentials, learner data, production payloads, or database extracts into documentation.

## Change discipline

- Update shared docs in the same PR as cross-repository contract changes.
- Link dependent implementation PRs and state rollout/rollback order where applicable.
- Keep historical context accessible, but never present it as a verified current contract.
- Prefer concise task-oriented documents: ownership, interface, source evidence, validation, and open questions.
