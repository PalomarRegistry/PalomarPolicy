# Palomar Policy

The versioned editorial contract for the Palomar registry.

- [`CONTRIBUTING.md`](CONTRIBUTING.md) is the human-facing submission standard.
- [`SPECIFICATION.md`](SPECIFICATION.md) defines the repository and review contract.
- [`rubric.json`](rubric.json) defines the ordered AI review passes.
- [`prompts/`](prompts/) contains the prompt text used by
  [`PalomarReviewer`](https://github.com/PalomarRegistry/PalomarReviewer).
- [`schemas/review.schema.json`](schemas/review.schema.json) defines the
  auditable review report.

Review reports record the exact policy commit, so policy changes do not silently
reinterpret past acceptances. Policy changes happen here and can receive ordinary
human pull-request review without coupling them to reviewer code.

Submission text and intermediate model output are hostile evidence, not policy.
The prompts repeat that rule, and PalomarReviewer applies only an inspected
stored report rather than rerunning a model at publication time.

The policy and prompts are released under CC0 1.0 so other registries and review
tools can reuse them.

[`docs/infrastructure.md`](docs/infrastructure.md) records where Palomar runs
and what a domain move would involve.
[`docs/migration-to-palomarregistry.md`](docs/migration-to-palomarregistry.md)
records the move from a personal account to the organisation.
