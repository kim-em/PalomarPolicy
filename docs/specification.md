# Palomar protocol

This document defines the current repository, review, registration, and
publication contract. `CONTRIBUTING.md` is the submitter-facing standard;
`rubric.json`, the files under `prompts/`, `schemas/review.schema.json`, and
`schemas/registry-correction.schema.json` are the machine-readable editorial
and correction-decision contracts;
`taxonomies/classification-guide.md` supplies the binding interpretation for
classification review. Every review records its exact PalomarPolicy commit, so
later policy changes do not reinterpret an earlier outcome.

## Components and authority

| Component | Authority | Must not do |
| --- | --- | --- |
| `PalomarServer` | Intake, GitHub push-access check, private status access, withdrawal, and registration consent | Execute source, run editorial review, or write registry records |
| `PalomarSubmission` | Public mechanical verification and registrable-Challenge rendering | Hold private identity or review state, or decide editorial policy |
| `PalomarSubmissionState` | Private append-only submission state and scheduled orchestration | Publish unregistered review outcomes or define policy |
| `PalomarPolicy` | Submission rules, rubric, prompts, and review and correction-decision schemas | Hold submissions, registered records, or credentials |
| `PalomarTemplate` | Reusable submission starter and CI example | Define binding policy or registry truth |
| `PalomarReviewer` | Automated editorial review, deterministic correction decisions, evidence validation, source preservation, registration PR preparation, clean-check merge, and finalization | Change policy, register without consent, or treat model output as trusted instructions |
| `PalomarDatabase` | Private canonical ledger, immutable record schemas, and filtered-publication tooling | Perform intake, execute source, or conduct review |
| `PalomarArchive` | Public native forks and immutable record-specific preservation tags | Hold private submission data or credentials that can mutate `PalomarRegistry` |
| `PalomarWeb` | Read-only human presentation of the public projection | Become another registry authority |

The public website and data service are views. The private PalomarDatabase
repository is the canonical ledger, and the private PalomarSubmissionState
repository is the canonical submission history.

## Submission lifecycle and privacy

```text
form or API submission, and push access proved
  -> verifying
     -> verification-failed
     -> awaiting-review
        -> reviewing
           -> review-failed
           -> review-ready
              -> registered | withdrawn
```

The internet-facing server is stateless between requests. It writes every
durable fact and transition to PalomarSubmissionState. Push access to the
submitted repository is established in one of two ways, and the private
submission record says which.

A person signs in with GitHub OAuth, and the server reads `permissions.push`
with that token before discarding it. GitHub answered for the same account that
authorised Palomar, so this establishes that one account both can push and
identified itself.

An agent, which has no browser, instead creates a tag at the submitted commit
and a secret gist carrying the same challenge, and Palomar reads both. Creating
a ref requires the same write access; the gist supplies an identity, because a
ref records no author and a third party cannot ask GitHub who has push. This
establishes that someone who can push submitted the repository and that an
account named itself, which are not provably the same account. It is
deliberately the weaker of the two, and the submission record carries the
distinction rather than implying parity. The published registry record does not:
it names no proof route, because it names no submitter either.

Push access is not proof of authorship or source-author approval either way, so
the submission separately records its authorization relationship.

The repository, commit, submission identifier, public verification run, and
mechanical logs are public from verification onward. The editorial review and
its outcome remain private unless the submitter registers a review that
identified no blocking problem.
Palomar publishes no submitter: no published record identifies the account that
proved push access, no schema has a field for it, and registration does not add
one. What is published about that person is the declared authorization
relationship and its optional free-text evidence, so a submitter who names
themselves there has published that much. “Private” means access-controlled,
not confidential: operators,
GitHub, and the model provider can see the material relevant to their roles.
Private submission state and delivered reviews are retained indefinitely so a
review outcome can be audited later.

The submitter may withdraw from any non-terminal state. Withdrawal leaves no
public editorial outcome. `verification-failed` means the mechanical gate did
not pass; `review-failed` means an operational review failure rather than an
editorial rejection. A `revision_required` or `rejected` outcome is not
registered and a corrected commit enters as a new submission.

## Mechanical verification

PalomarServer dispatches the public verification workflow and permanently pins
the matching run when it first observes it. The workflow publishes a
submission-scoped `mechanical-report.json`; that artifact, not the run title or
moving branch state, is the authoritative mechanical result.

A passing report binds at least:

- the submission identifier, source repository, full commit, selected project,
  requested paths—including the explicitly selected Comparator configuration
  path—workflow run, and workflow revision;
- the resolved and hashed Challenge, Solution, Comparator configuration,
  `formalization.yaml`, Lakefile, root licence, and Lean toolchain;
- the compared declarations, classifications, provenance, authorization-related
  source metadata, full dependency graph, and Challenge source closure; and
- the exact Comparator, lean4export, NanoDa, Landrun, and other trusted tooling
  revisions used by the run.

Every exported proof is checked by Lean's kernel and replayed through the
pinned independent NanoDa kernel. Palomar has no single-kernel passing result.
The submitted `enable_nanoda` field is optional and non-authoritative. Palomar
does not reject a repository because the field is absent or false; it writes a
separate protected configuration with NanoDa enabled and uses that for every
verification. This is intentional policy: independent replay is Palomar's
responsibility, not a configuration burden or veto delegated to submitters.

`PalomarSubmission/toolchains.json` is closed to exactly the fields
`schema_version` and `minimum`; it neither selects nor pins trusted tools. The
verifier derives the `lean4export` release tag from the submitted Lean version,
resolves that tag once to an exact commit, and records the commit; the renderer
does the same for Verso. Comparator, NanoDa, and Landrun use fixed verifier
pins. The mechanical and render reports record every exact trusted-tool
revision used for the run.

Mechanical metadata requirements are hard failures where `CONTRIBUTING.md` says
so. Palomar adopts `formalization.yaml` v0.4; current files should declare
`version: v0.4`, while an omitted version follows the upstream current-version
default. Provenance requires a nonempty
`project.responsible_maintainers` list whose members are nonempty name strings, a
nonempty source list with recognised relationships, and a valid optional
thin-wrapper mapping. An omitted `repository` defaults to the submitted
repository as the substantive development. A separately recorded
`substantive_formalization` requires an `owner/repository` or GitHub URL plus a
full 40-character lowercase commit SHA, which the verifier resolves after
validating its shape. Optional source types use the closed
vocabulary `paper`, `book`, `web discussion`, `folklore`, `original-proof`, and
`other`; `web discussion` is Palomar's canonical spelling within upstream
v0.4's open source-type field.

Exactly one origin is valid. An original result has at least one
`original-proof` source, each such source has relationship `other`, and all
relationships are `background` or `other`; the report records
`result_origin: original`. A source-based result has no `original-proof` and at
least one `formalizes`, `adapts`, or `independently-proves` relationship; the
report records `result_origin: source-based`. An undeclared or conflicting
origin fails. For compatibility with older files, the verifier accepts
`project.responsible_maintainer` and `sources[].author` as aliases when the
corresponding current plural key is not present at all; either alias may contain
one person or a list. A present plural field takes precedence even when empty or
invalid. An obsolete top-level `provenance` block is ignored, and cannot replace
the current source declarations or override their derived origin. The
mechanical report records maintainers and source authors under the current
plural names, preserves optional source contributors as `name`/`role`
records, and records the source-derived origin. These structural checks do
not replace editorial assessment of the facts' accuracy or the citations'
adequacy.
`automation.methods` remains required by the adopted upstream format, including
`method: manual`; `review.status` describes review performed before submission.
The schema can establish only that author and responsible-maintainer names are
nonempty. The metadata review supplies the semantic gate: those two identity
fields are reserved for humans, and naming an automated system in either
requires correction before registration. AI contributions remain reportable in
automation metadata and narrative provenance.

Challenge and Solution must be distinct module names. Palomar asks Lake for its
ordered source paths and selects the first matching regular, non-symlink file
inside the selected project. The Challenge is compiled separately against a
frozen trusted environment, and every source in its transitive closure must
resolve to Lean core or the accepted Mathlib, Tau Ceti, and CSLib statement
surface.
Proof-only dependencies used by the Solution may be broader, but remain pinned
and confined.

The reviewer independently downloads the pinned artifact, checks that its
workflow revision remains in trusted history, and binds it to the private
submission record. Missing, stale, ambiguous, or malformed evidence fails
closed.

## Editorial review

No person starts an ordinary review or signs off on its outcome. The scheduled
private pipeline checks out the immutable source and recorded policy commit,
constructs the ordered evidence packets, invokes the configured model engine,
and validates every result against the rubric and schemas.

Each pass receives only its prompt, binding policy inputs, the mechanically
recorded submission evidence, and earlier pass results declared by the rubric.
Repository content, external material, tool output, and earlier model output are
untrusted evidence, never instructions. Every engine runs inside a fail-closed
Bubblewrap namespace that exposes the submission read-only, makes its dedicated
output directory the only host-backed writable bind, provides an empty
ephemeral home and temporary directory, and omits the runner's GitHub, database,
archive, and other operator credentials and files. That outer namespace is not
a tool sandbox: Codex keeps its inspection and shell tools under its own
read-only sandbox, but this runner never enables Codex web search and ignores
user configuration that could enable it. Claude has no explicit tools in
ordinary passes and only `WebSearch` and `WebFetch` in the literature and
notability pass; a configured custom command is arbitrary code inside the outer
namespace.

Codex and Claude must authenticate their model transport. Production review runs
the Codex engine, whose transport authenticates through a loopback credential
broker: the real provider key is held by a short-lived process outside the
engine namespace, and the namespace receives only a random per-pass capability,
delivered to Bubblewrap over a file descriptor rather than a command line or an
inherited environment. That capability is loopback-only and worth nothing once
the pass ends. The broker serves exactly one route and one configured model,
forwards only allowlisted request and response headers in each direction, and
refuses stored, background, continued, priority, and provider-hosted-tool
requests. It caps the pass's request count and its concurrency, limits each
request and each response by size, and stops admitting new requests once
cumulative tokens or estimated spend reach their thresholds; requests already in
flight may carry the pass past those two. There is no direct-authentication
fallback: a Codex pass that cannot reach its broker refuses to start rather
than using a reusable key.

The namespace still shares the host network, because the transport has to reach
that loopback listener. The Claude engine has no broker of its own; it binds a
reusable provider login into the namespace, so it refuses to run unless an
operator sets `PALOMAR_ALLOW_UNBROKERED_CLAUDE=1`, which no production workflow
sets. A custom command receives no provider credential at all. Every pass is
schema-validated and refused before reuse or release if its plain output matches
the configured key or an API-key shape. That output check remains a backstop,
not containment against encoding or another channel.

Synthesis must reproduce the evidence-check scores exactly. A clean check cannot
score below the rubric minimum, a score of 1 or 2 requires a failed check, and
a no-blocking-problem outcome cannot override any failed check. A non-mandatory
score of 3 may accompany a non-blocking warning when its material finding is disclosed;
notability remains a mandatory floor and failure there requires rejection
rather than revision. A review that identifies no blocking problem cannot
request changes, and a `revision_required` outcome must request at least one.
These are structural guarantees; Palomar
does not separately claim that a human confirmed the model's substantive
judgments.

The complete review remains private and is bound to the submission by digest.
Each check separates author-facing material findings from `internal_notes` that
record positive checks, excluded edge cases, and non-material concerns. The
status page states whether the automatic review identified blocking problems
and presents the author-facing prose. It does not expose the machine-readable
outcome, scores, evidence-check records, finding severities, or audit notes. For
a registered result, Palomar publishes a redacted archived review containing
the no-blocking-problem outcome and every material finding, but not scores, per-finding
severity, or `internal_notes`. The private canonical materials retain the
information needed to reconstruct how the outcome was reached.

There is no ordinary appeal or human override. A changed review must be produced
under the current contract and delivered again; registration consent never
carries across review bytes.

Intake does not yet deduplicate unregistered attempts or retain prior attempts
for the reviewer. Resubmitting the same commit can therefore obtain a fresh
sampled editorial outcome. A commit already registered for the same stable
result identity cannot become another registered version. Whether to prevent
repeat reviews before registration, expose attempt history, impose a cooldown,
or explicitly permit them remains an open integrity and abuse-policy question.

## Registration, versions, and publication

Only a review that identified no blocking problem can be registered. The
submitter explicitly consents to the digest of the delivered review. Before any
public registration side effect,
PalomarReviewer verifies the consent and evidence bindings and reserves the
permanent identifier and version in private state. A retry reuses that
reservation.

For a new result, the identifier contains the first registration date and the
next
six-digit serial free on that date, counting from 1. The serial was drawn at
random until 2026-08-07, to hide how many reservations never became records.
What that cost was ordering: with a random serial the registration order of two
identifiers cannot be read from the identifiers, so every surface wanting that
order had to carry an ordinal beside the identifier, and an ordinal and an
identifier that disagree is a failure nothing downstream can detect or repair.
A correction or dependency update whose review identified no blocking problem
creates the next
integer version of an existing identifier; a new mathematical result receives a
new identifier. Explicit version URLs are immutable, while an unversioned URL
resolves to the latest active version.

Every version independently proves write access to its submitted repository and
records the submitter's current authorisation relationship. Citing an existing
identifier is routing, not authority. Automated updates must retain the stable
source repository, selected project path, and Comparator configuration path;
repository transfers require operator review. A source commit already present
in that identifier's version history cannot be registered again.

The sole exception is an exceptional registry metadata correction authorized
by an active Palomar Technical Maintainer. It appends a version at the exact
current repository, commit, project path, metadata path, and Comparator path;
ordinary duplicate-commit rejection is therefore intentionally replaced by an
exact-baseline check. Only public descriptive metadata may change. The new
record carries a public explanation and changed-field list, attributes the act
to “Palomar / Registry correction”, inherits the baseline mechanical, render,
preservation, trust, review, and score evidence, and records a deterministic
correction decision rather than running a new model review. The maintainer
consents to the exact correction decision before registration. This correction
is registry housekeeping, not approval or endorsement of the project.

The protected browser path establishes Technical Maintainer membership from a
GitHub OAuth identity. The agent path establishes the same authority from the
GitHub-set owner of a fresh, secret, challenge-bearing gist. In either case the
owner's numeric GitHub id must be in the checked-in Technical Maintainer
allowlist. A correction does not require a source-repository tag or write
access: its exact-baseline binding prevents the source or registered paths from
moving, and repository capability is not the authority being asserted.

The append-only guarantees below have been in force since 2026-08-10, when
`.palomar-launched` was added to PalomarDatabase at the public-history boundary
commit `7c2f0db8`. Every Database change after that commit is checked against
them. Palomar has not launched in the sense of inviting submissions, but the
database is no longer disposable: a registration made now is permanent
publication history, and a record leaves public view only by the moderation
route described below.

After the submitter's consent is recorded, registration renders the registrable
Challenge, validates the render and complete record, archives the mechanical
report, normalized workflow provenance,
redacted review, and source-preservation receipt in one content-addressed
evidence tree, then opens a PalomarDatabase pull request. The private scheduler
merges only when GitHub reports the change clean and all database checks have
passed. Merging that pull request is the registration event. A later scheduled
step verifies the merged entry and finalizes the private submission state.

After the launch boundary, canonical entries, schemas in use, renders, and
evidence are immutable. The private database may record a moderator-authorized
suppression of one exact version from the generated public projection without
deleting or rewriting canonical bytes. The public service then omits the
affected entry and artifacts and serves only a minimal date-only tombstone.
One of the named [Moderators](governance.md#moderators) must authorize the
suppression, and a Technical Maintainer executes it through the validated
private-database workflow. The authorizer and private reason are recorded.
There is no submitter-facing ordinary takedown route: registration is not
reversible on request, and a submitter who wants a record changed registers a
further version. A data-protection or copyright request is not that, and it has
a route: [lawful requests](lawful-requests.md) states where to send one, who
decides it, and what it can obtain. The immutability and preservation promises
in this document, including the source-preservation promise below, are
immutability against ordinary mutation; they do not displace applicable law,
and a Moderator may authorize the narrow exceptions that document describes.

The public data service is an active-only projection built from the private
ledger. A record is copied into it byte for byte rather than rewritten, so its
published bytes are a function of the commit and not of the publisher; anything
the public must not see is kept outside the record in the first place.
Publication writes each new record once at a key that never changes, writes the
aggregates under the new release, verifies every digest by reading it back, and
atomically advances the current-release pointer last. The read-only Worker
exposes only allowlisted paths and has no raw-GitHub fallback. There is no
whole-registry document: a reader takes what is new, one result's versions, a
day of the archive, a classification code, or a search word, and each of those
costs what it answers rather than what the registry holds. PalomarWeb consumes
this projection and is never a source of registry truth.

## Source preservation

Before a registration branch is published, PalomarReviewer preserves the
submitted repository, every pinned Git dependency, and any separately recorded
substantive formalization. Each must be a public GitHub repository pinned to a
full commit and representable by an ordinary native fork. Git LFS is rejected
throughout the graph. Submitted and substantive repositories may not contain
submodules; an inert dependency gitlink is permitted only because verification
never initializes it and the ordinary Git object is preserved.

Repositories in one GitHub fork network share one public fork owned by
`PalomarArchive`. Every registered commit receives an immutable record-specific
tag protected by a no-bypass ruleset. The archive identity is demoted from its
temporary per-fork administrator grant before the tag is written, and every
fork, commit, tag, and receipt is read back. Any failure stops registration.

The record and `source-archive.json` bind the complete source graph to those
forks and tags. Availability monitoring checks both original and preserved
commits. The intentional preservation promise is this maintained native-fork
copy; Palomar does not additionally archive Lean toolchain binaries, Mathlib
cache objects, or other non-Git artifacts.

## Security and deferred boundaries

Submitted source is hostile. Verification and rendering execute without
credentials or network access inside Landrun, with ceilings on time, memory,
processes, files, and output. Palomar never runs `lake update` or submitted
post-update hooks. Resource exhaustion is a retryable infrastructure outcome,
not a mathematical rejection.

The server holds separate least-privilege credentials for private state and
public workflow dispatch, so compromising the intake service does not grant
write access to verification code or the registry. Registry and archive
automation use distinct identities. The review namespace omits those
registration credentials and unrelated operator files, and the production
engine's provider key never enters it: the broker boundary described above
hands the namespace a per-pass capability instead. The credential-output scan
mitigates one obvious release route for the unbrokered engine path; it does not
replace that boundary.

Rendered HTML is sanitized, content-addressed, and served from
`data.palomar-registry.org`, separate from the website origin. PalomarWeb embeds
it with a restrictive CSP and an iframe sandbox without `allow-same-origin`.

The current service accepts only public GitHub repositories and Lean projects.
Private-repository support, DOI minting, automatic dependency updates, stronger
repeat-submission abuse controls, and proof assistants other than Lean remain
future work tracked outside this specification.
