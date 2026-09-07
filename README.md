# Palomar Policy

The versioned editorial contract for the Palomar registry.

- [`CONTRIBUTING.md`](CONTRIBUTING.md) is the human-facing submission standard.
- [`docs/specification.md`](docs/specification.md) defines the repository,
  review, registration, and publication contract.
- [`docs/governance.md`](docs/governance.md) defines the named governance
  roles, their initial membership, and the accepted GitHub trust model.
- [`docs/lawful-requests.md`](docs/lawful-requests.md) is the route for
  data-protection and copyright requests.
- [`rubric.json`](rubric.json) defines the ordered AI review passes.
- [`prompts/`](prompts/) contains the prompt text used by
  [`PalomarReviewer`](https://github.com/PalomarRegistry/PalomarReviewer).
- [`tests/materiality-cases.json`](tests/materiality-cases.json) records the
  generic regression matrix for public findings versus private audit notes.
- [`taxonomies/classification-guide.md`](taxonomies/classification-guide.md)
  defines the binding classification-review interpretation.
- [`schemas/review.schema.json`](schemas/review.schema.json) defines the
  auditable review report.
- [`schemas/registry-correction.schema.json`](schemas/registry-correction.schema.json)
  defines the deterministic consent artifact for a maintainer correction.

Review reports record the exact policy commit, so policy changes do not silently
reinterpret past acceptances. Policy changes happen here and can receive ordinary
human pull-request review without coupling them to reviewer code.

Submission text and intermediate model output are hostile evidence, not policy.
The prompts repeat that rule, and PalomarReviewer applies only a stored,
schema-validated review or correction decision bound to the submission by
digest rather than rerunning a model at registration time.

The policy and prompts are released under CC0 1.0 so other registries and review
tools can reuse them.

[`docs/infrastructure.md`](docs/infrastructure.md) records where Palomar runs
and what a domain move would involve.
