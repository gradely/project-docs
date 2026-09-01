# Gradely project documentation

This is Gradely’s central cross-repository architecture and integration reference.

## Start here

The **verified** documents cite the repository branch and source path used to establish each claim. They distinguish code-backed facts from items that need deployment or operational confirmation.

- [Architecture baseline and recommendation streak flow](docs/architecture-baseline.md)
- [Backend and service boundaries](docs/backend-service-boundaries.md)
- [Frontend and AI boundaries](docs/frontend-ai-boundaries.md)
- [Verification register and prioritised review backlog](docs/verification-register.md)
- [Legacy micro-frontend map](docs/legacy-microfrontend-map.md)
- [Branch, pull request, and ticket workflow](docs/branch-pr-ticket-workflow.md)

## Using this repository

- Use the verified documents to identify ownership and trace cross-repository changes.
- Keep implementation detail, commands, tests, and deployment configuration in the repository that owns the system.
- Update these documents in the same pull request as an API or cross-service contract change.
- Do not record credentials, learner data, database dumps, or production payloads here.

## Historical references

Earlier backend and frontend guides are preserved because they contain useful context. They are not current contracts and may include stale examples or operational assumptions:

- [Historical backend integration reference](docs/historical/backend-integration-reference.md)
- [Historical frontend micro-frontend reference](docs/historical/frontend-microfrontend-reference.md)

## Contribution guidance

See [AGENTS.md](AGENTS.md) for documentation verification and contribution rules.
