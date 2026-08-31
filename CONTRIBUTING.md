# Submitting to Palomar

Palomar is a registry of machine-checked formal proofs in Lean 4. It records an
exact commit of a public GitHub repository together with a small set of Lean
declarations that state the mathematical result.

The Lean evidence is organised as a Challenge/Solution pair. The Challenge
contains the declarations that state the result and is deliberately kept small
so that a mathematical reader can audit it. The Solution contains corresponding
declarations with the same types and supplies the proofs or definition values.

[Comparator](https://github.com/leanprover/comparator#comparator) checks that
the declarations in the Solution really are proofs, or definitions, of the
declarations stated in the Challenge, with the same types, using only the
permitted axioms. Palomar needs this comparison because the Challenge is the
small, human-auditable statement of record. Comparator makes it a mechanical
fact, rather than a claim, that the proof proves the stated thing.

A submission also has structured metadata and an informal mathematical account.
Palomar uses the upstream community
[formalization.yaml](https://github.com/mathlib-initiative/formalization.yaml)
self-reporting standard for formalisation projects; this is not a format
invented by Palomar. `formalization.yaml` records the required structured facts
about provenance, sources, licence, classification, authorship, automation,
review, any thin-wrapper relationship, scope, and known gaps. Its factual
`project.description` field supplies the public registry abstract. Fuller prose
explaining what the result says and why it matters may be in Challenge module
documentation, docstrings attached to the compared declarations, the project
README, `formalization.yaml`, or several of these locations. The formal
statement alone does not record all this context, and the editorial review
reads the submission as a whole against the Lean.

Before a submission can be registered, it must satisfy two kinds of check:

1. Mechanical verification found that Comparator accepts the recorded Solution
   declarations as implementations or proofs of the recorded Challenge
   declarations, using only the permitted axioms. The exported proof is checked
   by Lean's kernel and by the independent NanoDa kernel checker.
2. An AI editorial review identified no blocking problem in the alignment
   between each recorded formal statement and its informal description, or in
   the requirement that the result have a plausible research audience.

Palomar is a registry, not a journal. The automated review is a filter: it can
identify a blocking problem, or find no blocking problem, but it does not
accept, approve, or endorse a submission. Finding no blocking problem does not
claim novelty, independently validate an informal proof, or constitute review
by a human expert. It is also not registration. The submitter sees the review
first and decides whether to register it with the registry record.

Submissions go to [submit.palomar-registry.org][submission-server], and that is
the only way in. An agent submitting on someone's behalf should read
[`llms.txt`][llms] there before its first request: it documents the endpoints,
what each of them proves, and what registering publishes.

This document is the submitter-facing standard. Where it summarises what Palomar
publishes or what a submission proved, the binding statement is
[the protocol specification][specification], and this document defers to it.

[submission-server]: https://submit.palomar-registry.org/
[llms]: https://submit.palomar-registry.org/llms.txt
[specification]: docs/specification.md

## 1. Decide whether the result is suitable

Mechanical correctness is necessary but not sufficient. The mathematical
result, as it is actually stated in the Challenge source, must satisfy both of
these tests:

1. It could plausibly warrant a research paper or a serious research note.
2. The reviewer can identify a credible research area and a plausible kind of
   mathematician in a research department who could reasonably find it
   interesting or relevant.

Both tests are required. A specialised result may qualify when it has a
credible specialist audience. Formal correctness, the difficulty or size of the
Lean development, polished prose, and novelty do not by themselves establish
research interest.

Palomar does not index:

- trivial results presented as research contributions;
- theories whose definitions have been designed merely to manufacture the
  advertised conclusions;
- purported solutions of famous open problems without a careful comparison with
  the standard conjecture, a serious literature account, and an honest
  statement of any gap;
- duplicated or lightly repackaged work without useful provenance;
- deceptive, materially incomplete, or promotional metadata;
- submissions whose mathematical content cannot be identified from the
  Challenge and the informal account.

A clear failure of either research-interest test leads to `rejected`, not
`revision_required`.

## 2. Prepare an ordinary submission

The simplest submission is a public GitHub repository whose root is the Lean
project. In the ordinary layout, the Challenge, Solution, Comparator
configuration and formalisation metadata sit alongside the files that fix the
Lean toolchain and describe the Lake build and dependencies, plus a licence for
the submitted repository snapshot. Use these conventional files:

```text
lean-toolchain
lakefile.toml                 # or lakefile.lean
lake-manifest.json
formalization.yaml
Challenge.lean
Solution.lean
comparator.json
LICENSE                       # or another accepted licence filename
```

Submit the repository as `owner/name` together with the full 40-character SHA
of the commit to be reviewed, and explicitly select the repository-relative
Comparator configuration path (`comparator.json` in this ordinary layout).
Branch names and tags are not accepted as substitutes for a commit. The
checked-out repository, excluding `.git` and symbolic links, must be no larger
than 500 MiB.

One submission and one Palomar entry correspond to exactly one Comparator
configuration. If a repository/commit contains twelve different configuration
files, submit it twelve times with twelve different paths. Those become twelve
entries sharing a repository and commit but retaining distinct path and
declaration information. If one configuration selects many theorems or
definitions, it remains one entry and the reviewer audits every selected
declaration.

In this ordinary layout:

- `Challenge.lean` contains the declarations that state the result: the small
  statement file that a mathematical reader is expected to audit. Its module
  documentation and declaration docstrings may contain some or all of the
  narrative mathematical account.
- `Solution.lean` contains declarations with the same types and supplies the
  proofs or definition values.
- `comparator.json` tells Comparator which Challenge and Solution modules and
  declarations to compare.
- `formalization.yaml` gives the required structured provenance, sources,
  licence, authorship, process, classification, limitations, and review history,
  and its `project.description` is the concise public abstract. It may also
  contain more of the narrative mathematical account.
- the project README may contain some or all of the narrative mathematical
  account.
- the licence file states the licence for the submitted repository snapshot.

The filenames `Challenge.lean` and `Solution.lean` are conventions rather than
requirements. Alternative module names and less common layouts are described in
section 6, after the ordinary case.

### 2.1 Lean and Lake files

The Lean and Lake files fix the environment in which the submission is built.
The toolchain selects Lean, the Lakefile describes the project, and the
manifest records its exact dependencies. Together they allow the verifier to
reconstruct the submitted development at the pinned commit.

A TOML Lakefile is declarative and can be parsed without executing submitted
configuration. A `lakefile.lean` is executable Lean code, so without a
committed manifest the verifier cannot determine its exact dependencies before
running submitted code. This is why a Lean Lakefile always needs a committed
manifest.

#### Mechanical requirements

- The `lean-toolchain` file must name a Lean release, in the form
  `leanprover/lean4:<version>`, no older than the minimum recorded in
  `toolchains.json` in the [PalomarSubmission repository][submission-repo].
  That closed file contains exactly the fields `schema_version` and `minimum`.
  It neither selects nor pins trusted tools. The verifier derives the
  `lean4export` release tag from the submitted Lean version, resolves that tag
  once to an exact commit, and records the commit; the renderer does the same
  for Verso. Comparator, NanoDa, and Landrun use fixed verifier pins. The
  mechanical and render reports record every exact revision used.
- The project root must contain exactly one of `lakefile.toml` and
  `lakefile.lean`. The Lakefile must be a regular file no larger than 1 MiB. A
  TOML Lakefile must be valid TOML.
- Commit `lake-manifest.json`. It is mandatory for a `lakefile.lean` project.
  The narrow exception for a TOML project without a manifest is in section 6.3.

### 2.2 Challenge and Solution modules

The Challenge is the small statement file that a mathematical reader is
expected to audit. The Solution supplies the proofs or definition values. A
reader should be able to identify the exact mathematical result from the
Challenge without having to disentangle the proof development.

The Challenge should be short and readable:

- Prefer imports from Mathlib alone. Tau Ceti and CSLib are permitted, but
  Palomar records them as qualified dependencies and displays a warning about
  the larger body of code that must be trusted when reading the statement.
- Prefer theorem statements to new definitions.
- Give every definition needed by a compared theorem a precise docstring and
  its ordinary mathematical meaning.
- State every principal claim attributed to the formalisation. Do not hide
  material hypotheses, weaken quantifiers, replace a standard notion with a
  convenient surrogate, or present a merely supporting lemma as the advertised
  theorem.

Choose module names that remain unambiguous, and keep their source files in the
committed project source tree. Submitted `.lake` directories are discarded
before verification and must not be used to hold these files.

#### Mechanical requirements

- The Challenge and Solution modules must be distinct dotted Lean module
  names. Every component must match `[A-Za-z_][A-Za-z0-9_']*`.
- Palomar asks Lake for its ordered source paths and selects the first regular,
  non-symlink source file matching each module. Each selected file must lie
  inside the Lean project.
- The Challenge has a hard limit of 100 KiB and 1,000 lines. A Challenge over
  32 KiB or 300 lines receives a mechanical warning because it is harder to
  audit.

### 2.3 Comparator configuration

`comparator.json` identifies the Challenge and Solution modules, the
declarations to compare, and the axioms that the comparison may use. It
therefore fixes the exact formal claims tested by mechanical verification.

A name in `definition_names` identifies a definition whose value is left
unspecified in the Challenge and supplied by the Solution. If you use this
feature, explain the intended value and why the compared theorems constrain it.
Editorial review may reject a definition that makes the result vacuous even
when Comparator accepts it.

#### Mechanical requirements

`comparator.json` must be a regular JSON file containing one object and must be
no larger than 1 MiB. It has four required keys:

```json
{
  "challenge_module": "Challenge",
  "solution_module": "Solution",
  "theorem_names": ["MyProject.main_theorem"],
  "permitted_axioms": ["propext", "Quot.sound", "Classical.choice"]
}
```

- `theorem_names` must be a nonempty array. The optional `definition_names`
  array defaults to empty. Every entry in either array must be a nonempty
  string.
- No other keys are accepted, apart from the optional `definition_names` and
  `enable_nanoda` keys.
- `permitted_axioms` may contain only `propext`, `Quot.sound`, and
  `Classical.choice`.
- `enable_nanoda` is accepted for Comparator compatibility but is intentionally
  non-authoritative. Its submitted value is ignored, and the field may be
  absent. Palomar always writes a separate protected configuration with NanoDa
  enabled. Requiring authors to maintain this switch would add an intake
  barrier without changing what Palomar executes.
- Deliberate holes in Challenge declarations are allowed. Comparator must
  confirm that every proved Solution declaration does not depend on `sorryAx`,
  `Lean.ofReduceBool`, a custom axiom, or an unnamed missing definition.

### 2.4 Dependencies

The Challenge has a stricter dependency rule than the Solution because it is
the statement a reader must trust. Its **transitive import closure** means the
Challenge source and every Lean source file reached by following its imports
recursively. Every file in that closure must be one of:

- Lean core;
- Mathlib at a verified revision in its canonical repository, together with the
  exact dependencies pinned by Mathlib's manifest;
- Tau Ceti at a verified revision in its canonical repository, together with
  the exact dependencies pinned by Tau Ceti's manifest.
- CSLib at a verified revision in its canonical repository, together with the
  exact dependencies pinned by CSLib's manifest.

No other project-specific source may occur in the Challenge's transitive import
closure. Recursive imports are treated exactly like direct imports. Previous
registration by Palomar does not make a repository an approved Challenge
dependency.

Dependencies reached only from the Solution may come from any public GitHub
repository that satisfies the requirements below. This permits a proof to use
a broader development without making that development part of the statement a
reader must audit. The mechanical report records the whole project dependency
set separately from the smaller set used by the Challenge.

#### Mechanical requirements

- The project may use Git dependencies for its proofs. Every Git package in
  `lake-manifest.json` must use a credential-free public
  `https://github.com/owner/repository` URL without a query or fragment, and
  must be pinned to a full 40-character lowercase commit SHA.
- The submitted repository and any separately named substantive formalisation
  must not contain Git submodules. A dependency may contain an inert submodule
  gitlink only because Palomar never initializes or reads it and the native
  archive fork retains the exact gitlink.
- Git LFS pointers are rejected in the submitted repository, every dependency,
  and any separately named substantive formalisation. Registrable source must be
  fully present in ordinary Git objects so the complete consumed source graph
  can be preserved in native GitHub forks.
- Do not commit compiled Lean or native build output outside `.lake`. The
  verifier rejects files with compiled-artifact suffixes including `.olean`,
  `.ilean`, `.a`, `.bc`, `.dll`, `.dylib`, `.o`, `.obj`, `.so`, and `.trace`,
  and it replaces submitted `.lake` state with fresh build directories.

### 2.5 Repository licence

The repository licence declares the licence for the submitted repository
snapshot at the pinned commit. It does not change the licence of cited papers,
mathematical sources, reused formalisations, or dependencies. Palomar records
the declared and detected identifier, but does not verify ownership or provide
legal advice.

The licence file and `project.license` in `formalization.yaml` must identify
the same unambiguous standard SPDX licence.

#### Mechanical requirements

The repository root must contain exactly one conventional licence file. Its
name is case-insensitive and must be one of:

- `LICENSE`, `LICENCE`, `COPYING`, `UNLICENSE`, or `OFL`;
- one of those names followed by `.md`, `.markdown`, or `.txt`.

The file must be a regular, non-symlink, nonempty UTF-8 text file no larger
than 1 MiB. Palomar's licence detector must find exactly one unambiguous
standard SPDX identifier, such as `Apache-2.0`, and that identifier must match
`project.license` in `formalization.yaml` exactly. A missing, multiple, custom,
ambiguous, or mismatched licence fails mechanical verification before
editorial review.

## 3. Write `formalization.yaml`

`formalization.yaml` is the required structured mathematical and editorial
record of the submission. It gives a reader the facts needed to identify the
exact claim, understand its provenance and limitations, and assess how the work
was produced and reviewed. Its `project.description` is always required as the
concise public registry abstract. More detailed narrative need not be
duplicated there when it is supplied in an eligible Challenge documentation
location or README.

Palomar uses the [mathlib-initiative `formalization.yaml` v0.4
format](https://github.com/mathlib-initiative/formalization.yaml) as a base and
accepts unknown fields. Palomar makes the upstream subject-classification,
responsible-maintainer, and provenance fields mandatory and applies the tighter
registry rules below. Provenance is
carried by `project.responsible_maintainers`, `sources`, and, only for a thin
wrapper, `repository.substantive_formalization`, not by a top-level field. The
file may contain those additions at the same time as upstream fields such as
`status`, `fidelity`, and `alignment`.

For compatibility with older files, Palomar accepts
`project.responsible_maintainer` and `sources[].author` as aliases when the
corresponding current plural key is not present at all. Despite their singular
names, each alias may contain one person or a list. A present plural field takes
precedence even when empty or invalid, and the alias is ignored. The verifier
also ignores an obsolete top-level `provenance` block. These compatibility forms
cannot replace the current source contract: result origin is always derived
from `sources`. The mechanical report records maintainers and source authors
under the current plural names and records the source-derived origin. New files
should use the plural fields and omit the top-level block.

#### File requirements

The file must be UTF-8 YAML no larger than 256 KiB. It must contain one
top-level mapping. Duplicate mapping keys and YAML merge keys are not accepted.
Current metadata should declare `version: v0.4`; files without a version remain
readable because the upstream dispatcher treats an omitted version as current.

### 3.1 Fields checked mechanically

The mechanically checked fields identify the project, classify the mathematics,
and record how the formalisation was produced and reviewed. The values still
require mathematical and editorial judgement.

Classify the mathematics, not merely the use of Lean or AI. A code may describe
the statement, an ancillary result, or mathematics in the proof; it need not be
unique or optimal. Palomar presumes possible proof relevance without checking
it and rejects only unmistakably egregious mismatches. For example:

```yaml
classification:
  arxiv: [math.CO, math.NT]
  msc2020: [05C10, 11N13]
```

The `automation.methods` and `review.status` fields come from the upstream v0.4
self-reporting format. For portable metadata, choose `manual`, `copilot`,
`agent`, `autonomous`, or `other` for each automation `method`; for example,
record a broader description such as “AI-assisted” in `tool_setup` or the
automation notes. Palomar intake accepts unfamiliar nonempty method wording and
retains it in the submitted evidence, but it is not a new standard category.
For work performed without an automated system, use `method: manual`.
`review.status` describes the review completed before
submission, not the Palomar review that is about to occur. Upstream examples
include `unchecked`, `agent-reviewed`, `self-assessed`, `peer-reviewed`,
`author-verified`, and another accurately described free-form status. Use
`review.reviewers` when identifiable people or systems performed a distinct
review, and use `review.notes` for a concise basis or a pointer to a fuller
pinned account. Do not imply a separate review merely because the authors
checked their own work; `unchecked` is accurate when no review was performed.

Record each material automated method and model role honestly. Costs, hardware,
wall time, and prompt logs are useful when available but are not required to be
reconstructed after the fact. A concise structured disclosure may point to a
fuller account in a document at the reviewed repository commit.

Palomar reserves `project.authors` and `project.responsible_maintainers` for
humans; do not list an AI model, agent, system, session, or tool in either
field. Credit material AI contributions in `automation.methods` and the
narrative production account instead. This follows the
[Leiden Declaration's human-authorship principle][leiden-human-authorship]:
credit and responsibility remain with people even when automated systems make
substantive contributions. Because names cannot be classified reliably by a
schema, the metadata review enforces this rule editorially.

#### Mechanical requirements

These fields are hard mechanical requirements:

- `project.name`: a nonempty string of at most 300 characters. This is the
  default public entry title; the source repository is shown separately;
- `project.description`: a nonempty string of at most 10,000 characters. This
  is the exact abstract shown in the Registry and gives a concise account of
  the mathematical content and principal results of the formalization as a
  whole;
- `project.authors`: a nonempty list of nonempty name strings. Palomar still
  reads the former mapping form for compatibility, but new metadata should use
  the v0.4 string form. Every listed author must satisfy the human-only rule
  above;
- `project.license`: the exact SPDX identifier detected from the root licence
  file;
- `classification.arxiv`: one to eight distinct codes from Palomar's checked-in
  arXiv taxonomy snapshot, `taxonomies/arxiv-categories.json`, in the
  [PalomarSubmission repository][submission-repo];
- `classification.msc2020`: up to eight distinct codes from Palomar's
  checked-in MSC2020 snapshot, `taxonomies/msc2020-codes.json`, in the
  [PalomarSubmission repository][submission-repo]; a submission that records no
  MSC2020 code is accepted;
- `automation.methods`: a nonempty list of mappings, each with a nonempty
  `method`;
- `review.status`: a nonempty string.

### 3.2 Provenance

Provenance tells the reader where the result came from, whether this repository
contains the substantive development, and who is responsible for the submitted
formalisation. These questions affect how the mathematical claim and its
relationship to earlier work should be assessed.

The required `sources` list records the result's origin. Use a source entry with
`type: original-proof` only when the formalisation is the first presentation of
the result; its title should identify that result in human-readable terms and
its relationship must be `other`. An original result may list additional
background material with relationship `background` or `other`. Otherwise,
record the work presented elsewhere with a relationship of `formalizes`,
`adapts`, or `independently-proves`. A new proof of a known result is
source-based and uses `independently-proves`; it is not an `original-proof`.
Prior work that the recorded result substantively formalises or adapts likewise
makes the result source-based. In original mode, use `background` only for
contextual material; use `other` with an explanation when the closed vocabulary
has no more accurate non-substantive relationship.

#### Mechanical requirements

Mechanical verification requires this current shape:

- `project.responsible_maintainers`: a nonempty list of nonempty name strings.
  Palomar still reads the former mapping form for compatibility, but new
  metadata should use the v0.4 string form. Every listed maintainer must satisfy
  the human-only rule above. For a thin wrapper these are people
  responsible for the submitted wrapper; the submission's authorisation
  relationship separately covers the underlying substantive formalisation. The older singular alias is
  accepted only as the compatibility fallback described above;
- `repository` is omitted when the submitted repository contains the
  substantive development; the mechanical report records that ordinary
  default as `substantive-development`. A thin wrapper must provide
  `repository.substantive_formalization.id` as `owner/repository` or a
  `https://github.com/owner/repository` URL and
  `repository.substantive_formalization.revision` as a full 40-character
  lowercase commit SHA. This first validates and normalises the metadata shape;
  the verifier later resolves the named GitHub repository and exact commit and
  fails if they cannot be used as the substantive source. The older explicit
  `repository.role` spellings remain accepted: `thin-wrapper` requires the
  mapping, while `substantive-development` forbids it;
- `sources`: a nonempty list in which every entry has a nonempty `title` and a
  `relationship` of exactly `formalizes`, `adapts`, `independently-proves`,
  `background`, or `other`;
- optional `sources[].contributors` is a list of mappings, each with a
  nonempty `name` and a nonempty, at-most-200-character free-form `role`;
- when a source has a `type`, it must be `paper`, `book`, `web discussion`,
  `folklore`, `original-proof`, or `other`; the field may otherwise be omitted.
  The current Palomar spelling is exactly `web discussion`, with a space.

The source list must satisfy exactly one of these alternatives:

1. **Original:** at least one entry has `type: original-proof`; every such entry
   has `relationship: other`; and every source relationship is `background` or
   `other`. The report derives `result_origin: original`.
2. **Source-based:** no entry has `type: original-proof`, and at least one entry
   has relationship `formalizes`, `adapts`, or `independently-proves`. The report
   derives `result_origin: source-based`.

A source list that satisfies neither alternative fails mechanical verification:
for example, a list containing only `background` entries, or one combining an
`original-proof` with a substantive relationship. Missing fields, unrecognised
enumerated values, and an invalid thin-wrapper shape also fail. An ignored
legacy `provenance.result_origin` cannot supply a missing current source
declaration or override the origin derived by these rules.

These checks establish only that the required facts have a usable shape and a
consistent declared origin. Editorial review still assesses whether the named
people, repository relationship, citations, source relationships, and account of prior
work are accurate and informative. A structurally valid citation can therefore
still be inadequate or misleading.

### 3.3 Mathematical sources and related formalisations

A mathematical source may be a paper, book, web page, MathOverflow or another
discussion, private communication, or a folklore result. It is not a Lean
software dependency. Record previous formalisations separately in
`related_formalizations` so that a reader can distinguish mathematical sources
from earlier Lean work.

Give every source a nonempty `title` and the most stable identifier or location
available. Choose the relationship category that best describes how the
submitted result uses the source:

- `formalizes`: the Lean work formalises the source's result;
- `adapts`: it changes or extends the source's result;
- `independently-proves`: it proves the same result independently;
- `background`: the source supplies context rather than the recorded result;
- `other`: another relationship, which should be explained in `note`.

Palomar intake accepts an unfamiliar nonempty relationship rather than
rejecting the submission, but gives it the provenance semantics of `other`.
Portable metadata should use the canonical `other` spelling and retain the
submitter's exact description in `note`.

Source authors and identifiers may be omitted when they genuinely do not exist
or are unknown. Contact and endorsement are useful context but are not required
and do not replace submitter authorisation.

Use `authors` only for a source's bibliographic authors. Use `contributors`
for other named credits, such as `editor` or `problem-proposer`, with one
`name` and free-form `role` per entry. A solution paper and the collection
that posed its problem should normally remain distinct source entries so that
each retains its own authorship, contributor roles, identifier and relationship
to the formalization.

For each previous formalisation, use `note` to explain whether the present
work extends, reimplements, ports, compares with, or otherwise relates to the
earlier work.

#### Field constraints

- For portable metadata, `author_endorsement` uses
  `participated`, `endorsed`, `no-response`, `not-contacted`, `declined`, `n/a`,
  or `other`; explain `other` in the source `note`. Palomar accepts unfamiliar
  wording as evidence but does not turn it into a categorical endorsement.
- Each entry in `related_formalizations` must have an `id`. Portable metadata
  uses `builds-on`, `adapts`, `independent`, `supersedes`, or `other` for its
  relationship and the existing `note` field for nuance. Palomar accepts an
  unfamiliar relationship but gives it `other` semantics at registration.

### 3.4 The informal account

Write for a mathematically literate reader outside the immediate project. The
fuller narrative mathematical account may be in Lean module documentation in
the Challenge source, docstrings attached to the compared declarations, the
selected-project README or repository-root fallback, or `formalization.yaml`.
It may be in one of these locations or divided across several, and need not be
duplicated. `project.description` is the one required synopsis: it identifies
the mathematical subject and principal result families, but it is not a
declaration manifest and need not enumerate every theorem, variant, definition,
hypothesis, or constant. Taken together, the supplied prose must make it
possible to identify and assess the exact claim being submitted.

During preliminary checks, the submission page displays the exact registry
abstract and the selected Comparator declarations even when every check passes.
It remains a preview unless the submitter chooses to edit it; after
authentication, Palomar can then open a pull request changing
`project.description` in the submitted repository.

Keep all required structured facts in `formalization.yaml`, including
provenance, sources and their relationships, licence, classification,
authorship, automation, review, any thin-wrapper relationship, scope, and known gaps.
Narrative elsewhere supplements those fields and does not replace them.

Across the eligible narrative locations, include:

- a plain-language account of every compared theorem;
- every known mismatch with the cited source, extra assumption, permitted
  axiom, scope restriction, degenerate case, and other limitation;
- an explanation of the mathematical sources used to choose, state, adapt, or
  justify the result, consistent with the precise references and relationships
  recorded in `formalization.yaml`;
- what is original, translated, adapted, proved, or still missing;
- the relation to previous formalisations;
- the authorship and production process, including AI involvement and human
  review;
- the repository licence.

The statement-alignment review treats the required description as a concise,
project-wide public abstract. The selected Comparator configuration is evidence
for checking that the abstract at least points, directly or collectively, to
the mathematical subject and principal result families represented by that
selection. The abstract may accurately describe additional project results,
including results checked by another configuration. The review creates a
finding only when the abstract is materially false or misleading, unrelated to
the selected work, or omits a distinct principal result family represented by
the selection—not because an individual declaration is not separately
identifiable. Presence, shape, and length are checked mechanically before
editorial review.

Do not claim novelty without a credible literature search. If novelty has not
been established, say that it is unknown.

A source does not have to be archivable. Mathematics is communicated in
preprints, talks, social media posts, private correspondence and folklore, and
review judges the account you give, not the medium. Where a source cannot be
independently confirmed, say so, give the most stable identifier that exists,
and claim no more than it supports; you will not be asked to produce an archive
that does not exist.

Such a source is not evidence, though. Palomar records it as your account of
where the result came from, and it counts towards nothing else: novelty,
priority and notability have to stand on the result and on what can be checked.
What is marked down is a material citation that is wrong or
misattributed, a material claim resting on a source you do not identify, or
novelty claimed with no search behind it.

An informal account of the proof is optional and may be supplied in any of the
eligible narrative locations. If supplied, it must describe the architecture
and decisive steps of the Lean proof that is actually present, including
important assumptions and computational components. The reviewer compares it
with the Solution source; a plausible proof of the same theorem is not enough.

[submission-repo]: https://github.com/PalomarRegistry/PalomarSubmission
[leiden-human-authorship]: https://leidendeclaration.ai/#human-authorship

## 4. Confirm that you are authorised to submit

You must either:

- be a responsible author or maintainer of the substantive formalisation; or
- have approval from one.

The submission form records which basis applies. A link or short note
documenting approval is optional. For a thin wrapper, the relevant person is
responsible for the underlying formalisation repository, not merely the wrapper
repository.

Answering that you are a responsible author or maintainer is itself the basis,
and review will not ask you to document approval from yourself. Write access,
a shared owner, organisation membership, a fork, and a transferred repository
are none of them that answer: they say what you can do, not what the work is
or whose it is. Answering falsely is a material misrepresentation.

Source-author contact or endorsement does not replace this authorisation.

Write access to the submitted repository is proved separately from this
question, and in one of two ways: a browser sign-in, or, for an agent with no
browser, a tag at the submitted commit together with a gist. The two do not
establish the same thing, and the private submission record says which was used;
[the protocol specification][specification] states the difference and is the
binding version of it. Neither is evidence of authorship, which is why you are
asked about the relationship here instead.

## 5. What mechanical verification establishes

The verifier checks out the exact commit and records hashes for the files it
uses. It validates the repository structure, metadata shape, licence,
dependency pins, Challenge dependency set, file sizes, and Comparator
configuration. For a thin wrapper it also resolves and inspects the named
substantive repository at the exact declared commit. It then:

1. discards submitted Lake build state and materialises the exact dependencies
   in the manifest;
2. compiles the Challenge separately against Lean core and the verified
   Mathlib, Tau Ceti, or CSLib dependencies;
3. records every Lean source file used by that compilation and rejects any
   source outside the permitted set;
4. protects that compiled Challenge module from replacement by project build
   output;
5. writes a protected Comparator configuration with NanoDa enabled outside
   every sandbox-writable directory;
6. runs Comparator without network access or credentials using that protected
   configuration; and
7. requires every exported proof to pass Lean's kernel and replay through the
   pinned NanoDa kernel.

A passing mechanical report establishes that the recorded Solution satisfies
the recorded Challenge under those checks. It does not establish that the
Challenge says what the metadata claims, that a definition has its ordinary
mathematical meaning, that the metadata is accurate, that the result is novel,
or that the result is interesting. Those are editorial questions.

The verifier distinguishes a submission failure from a retryable infrastructure
or resource error. Editorial review normally begins only after the mechanical
report passes.

## 6. Less common cases

Use this section only if the ordinary root layout in section 2 does not fit the
repository.

### 6.1 A Lean project below the repository root

The submission form can select one repository-relative directory as the
**selected project**. That directory becomes the root used for its Lakefile,
default metadata path and module resolution. The Comparator path is always
selected explicitly, even when it is the conventional `comparator.json`.

If the selected project has its own `lean-toolchain`, that file is used.
Otherwise the repository-root `lean-toolchain` is used. When both exist, the
project-local file takes precedence. The single licence file always remains at
repository root.

### 6.2 Explicit metadata and Comparator paths

The Comparator path is required. You may additionally supply the metadata path
when the selected file does not use its default:

- the metadata path may point anywhere inside the repository, but its basename
  must be exactly `formalization.yaml`;
- the required Comparator configuration path must point inside the selected project and
  must end in `.json`.

Every supplied path is relative to the repository root and uses `/`. It must
not be absolute or contain an empty component, `.` or `..` component,
backslash, query or fragment character, control character, drive prefix, or
symbolic-link component. The resolved object must be a regular file, or a
regular directory for the selected project, inside the checked-out commit.

### 6.3 A TOML project without its own manifest

If a `lakefile.toml` project has no `lake-manifest.json`, the verifier can
construct a temporary manifest only when every direct Lake dependency is a
contained path dependency, each target has exactly one regular Lakefile and a
committed valid manifest, none of those target manifests contains another path
dependency, package names do not overlap, and all targets use the same
contained packages directory.

Commit the selected project's own manifest for every other layout. This
includes any direct Git dependency and any overlap among package names
contributed by the path-dependency manifests. A `lakefile.lean` project always
needs a committed manifest for the reason given in section 2.1.

### 6.4 Contained path dependencies

A local Lake dependency may point to another directory in the same repository
checkout. The target must be a distinct regular directory, must not lie below
`.lake`, and must not be reached through a symbolic link or escape the
checkout. Its manifest's `packagesDir` must identify a contained directory
named `.lake/packages` owned by the selected project or a contained path
dependency.

The registered record normalises each path dependency to a
repository-root-relative directory. `.` in that record means the repository
root, regardless of how the Lakefile spelled the relative path.

### 6.5 Thin wrappers

A **thin wrapper** is a repository that exists only to expose declarations from
another formalisation to Comparator. Ordinary projects omit `repository`;
wrappers provide:

```yaml
repository:
  substantive_formalization:
    id: owner/repository
    revision: 0000000000000000000000000000000000000000
```

The repository must be a public GitHub repository, and `revision` must be a
full lowercase commit SHA. Palomar records this underlying repository and
commit as the substantive formalisation. Submitter authorisation must also
concern that underlying project. The legacy explicit `role: thin-wrapper`
spelling remains accepted but is unnecessary.

## 7. Editorial review

The review is performed by a language model working through a fixed sequence of
prompts. No person reads an ordinary submission before the outcome, and no
person signs it off. It is not peer review, and it is not evidence that a
mathematician has checked the result. It is a structured, recorded, automated
filter, and it should be weighed as that.

The review consists of required evidence checks followed by a synthesis step. A
check examines one subject and returns an outcome, findings tied to files or
other evidence, and one or more scores. Synthesis combines those fixed check
results into the final review outcome; it does not raise or average scores.

Every substantive check must return a coverage manifest containing every
theorem name and then every definition name in the recorded Comparator
configuration. The reviewer rejects an incomplete or reordered manifest. Clean
declarations need no individual comment, but every distinct material criticism
must be reported; finding one problem must not suppress review of later
declarations. A classification check likewise records every submitted code.
The reviewer mechanically requires the final AI-comment list to contain every
material finding from every evidence check, once and in order. A finding is an
author-facing criticism and may later be published. Positive checks, harmless
edge cases, excluded failure modes, and non-material concerns instead go into
private `internal_notes`; those notes cannot justify an outcome or requested
change. Severity remains internal; all findings are presented under the same
AI-comments heading. One Palomar
submission records one Comparator configuration. A
repository with several configurations must submit each configuration
separately if all of them are to become Palomar records.

The required checks examine:

- whether any arXiv or MSC2020 classification is egregiously off-topic, while
  presuming possible proof relevance without investigating it;
- the clarity, accuracy, and completeness of the required structured metadata,
  provenance, and narrative account across its supplied locations;
- alignment between every compared theorem and its informal account, including
  definitions, quantifiers, hypotheses, coercions, degenerate cases, and
  claimed scope;
- the fidelity and auditability of every material definition and imported
  concept used by the compared statements;
- the literature account and the result's research interest.

If an informal proof account is present in any eligible narrative location, an
additional check compares it with the actual Solution proof and its imports.

Each check uses one of three machine-readable outcomes:

- `neutral`: the check identified no material problem;
- `warning`: the check identified a specific non-blocking warning;
- `failure`: the check identified a material deficiency or contradiction, or could not
  affirmatively establish a mandatory criterion from the available evidence.

A failed check does not by itself choose the final outcome. Synthesis returns
`revision_required` when a specific, realistically correctable evidence or
presentation gap could make the submission qualify, and `rejected` for a
fundamental failure.
The report should describe uncertainty as an evidence limitation rather than
making a stronger negative claim than the evidence supports.

Scores run from 1 to 5:

- `1`: unusable, materially incorrect, or misleading;
- `2`: major errors or omissions;
- `3`: minimally adequate, but with meaningful limitations or unverified
  claims;
- `4`: thorough, fair, supported by evidence, and correct apart from minor
  issues;
- `5`: exceptionally complete and independently checkable, with no meaningful
  gap found after critical review.

The current rubric minimum is **4**. A score of 4 or 5 requires
concrete positive evidence, not merely successful compilation, populated
fields, familiar terminology, or the absence of an obvious contradiction. A
clean check must reach 4 on every score it owns. A non-mandatory dimension may
score 3 with a `warning` outcome and a concrete material finding without
blocking registration. A score of 1 or 2 is a failed check. Notability is
mandatory: below 4 it fails the check and requires rejection.

One exception, or these anchors would forbid what section 3 permits: a source
disclosed as unconfirmable, precisely stated, is not an "unverified claim" for
the `3` anchor, and does not by itself hold literature below 4. It cannot
support a `5`, which requires an account somebody else can check.

No score is published. They decide the outcome; the five above are kept in a
private file beside the database entry and the rest stay in the private review,
and the outcome is what a reader is shown, because the same repository at the
same commit has scored 5 and then 4 on one dimension across two runs of this
policy with the same outcome both times.

**No text a reader may see may state, bound, or imply a score.** That covers
every `summary` and every finding `message`. It is not enough to leave the
number out: "this prevents a literature score of 5 but not 4" names the score
exactly, and "every score meets the minimum" bounds all of them. Say what is
wrong or missing, and what would put it right. Write

> The unconfirmable source is disclosed precisely, and no priority is claimed
> from it.

rather than

> This prevents a literature score of 5 but not 4.

Both say the same thing about the submission; only the first says nothing about
the arithmetic.

Notability has its own anchors:

- `1`: incoherent, manufactured, materially deceptive, or framed in a way
  associated with mathematically baseless claims, with no credible
  contribution;
- `2`: identifiable but trivial, routine, lightly repackaged, or without a
  plausible research audience;
- `3`: borderline interest, where paper-worthiness or a credible research
  audience has not been affirmatively established;
- `4`: plausibly paper-worthy, with a specifically identified credible research
  audience;
- `5`: unusually consequential, with clear interest beyond a narrow specialist
  audience.

A notability score below 4 is a fundamental editorial failure and leads to
`rejected`, including when a credible research audience or plausible
paper-worthiness has not been affirmatively established. Findings may say
plainly that work is `trivial`, `confusing`, `unclear`, or `niche without an
identifiable research audience` where the evidence supports it. When the
evidence establishes only that the requirement was not demonstrated, the
finding should say that instead. Findings assess the work and its presentation,
never the submitter.

### Final outcomes

- `neutral`: the automated review identified no blocking problem. This permits
  registration but does not accept, approve, or endorse the submission. It may
  include disclosed, non-blocking warnings but no requested changes;
- `revision_required`: the result may qualify after specific, realistically
  correctable changes, which the review lists;
- `rejected`: there is a fundamental semantic, provenance, or editorial failure,
  including failure to affirmatively establish the research-interest
  requirement.

Palomar has no appeals route and no human sign-off on review outcomes. A submitter
who believes the reading is wrong should correct or strengthen the submission
and submit the corrected commit.

If an operator or tool failure prevents the automated review from completing,
the submission is marked `review-failed`. That is an operational state, not a
review outcome about the submission; Palomar may investigate and rerun it.

Every review includes a summary, author-facing findings, any requested
changes, private scores and audit notes, and a machine-readable report.

## 8. Privacy, registration, and rendering

The review and its findings are not public unless the submitter chooses to
register them. They are not secret either: they may be audited and acted on by
the Palomar moderation team, they pass through GitHub and the model provider,
and Palomar retains them indefinitely so that any review outcome can be examined
later.

Mechanical verification runs in a public GitHub Actions workflow, so the
repository, the commit, and the fact that they were mechanically checked are
public from the moment of submission. The mechanical report also includes the
declared authorization relationship and any optional approval evidence, so
those are public too. That workflow does only the mechanical check. It runs
before any editorial review, contains none of the review text, and shows no
review outcome, so its public log reveals nothing about whether a review
happened or what it found. The review and its outcome stay non-public unless the submitter
registers. Palomar publishes no submitter: the registry record has no field for
the account that proved push access, and registering does not add one. What is
published about that person is the authorisation relationship declared in
section 4 and, if you supply it, the free-text evidence beside it, which
identifies whoever you write into it.

On registration, Palomar archives a redacted copy of the review beside the
record; [the protocol specification][specification] says what that copy carries
and what is held back. It also creates
or reuses native public forks in
[`PalomarArchive`](https://github.com/PalomarArchive) for the submitted
repository, every pinned Git dependency, and any separately recorded
substantive formalisation. Every registered commit receives an immutable,
record-specific preservation tag. Registration stops before publishing a
database change if any source, fork, tag, or read-back check fails. The reviewer
model identifiers, exact source commit, mechanical workflow run, exact policy
commit, and source-preservation receipt also become public. Submitting grants
Palomar permission to quote the submitted metadata in the review report and
registry record.

Registering makes the redacted archived review, the source, and the record
public, and creates immutable source-preservation tags. The append-only record
guarantee has been in force since 2026-08-10, when `.palomar-launched` was added
to PalomarDatabase, so a registration is permanent publication history from the
moment it merges. Palomar has not launched in the sense of inviting
submissions, which is the only sense in which it is still pre-launch. A record
leaves public view only by moderation, which leaves a tombstone and ordinarily
keeps the canonical file; [`docs/lawful-requests.md`](docs/lawful-requests.md)
says how a data-protection or copyright request is made and decided, and when
applicable law requires more than that.

After the review identifies no blocking problem and registration consent is
recorded, Palomar renders the pinned Challenge source with Verso for display
before opening the database change.
Rendering compiles submitted Lean, so it runs
under the same restrictions as verification: no network access and no
credentials. The commit-pinned GitHub file remains the authoritative source. A
Challenge is eligible for inline display when exactly one declaration is
compared and the file is no more than 100 lines and 32 KiB. Larger
Challenges open in a dedicated rendered page. A rendering failure postpones
registration but does not change the review outcome.

## 9. Updates and permanent identifiers

When a new result is prepared for registration, Palomar assigns an identifier
of the form `PALOMAR-YYYY-MM-DD-NNNNNN`. The date is the first registration
date. The six-digit serial is the next one free on that date, counting from
`000001`. It was drawn at random until 2026-08-07, which hid how many
reservations never became records but also meant that the order two identifiers
were registered in could not be read from the identifiers, so every surface that
wanted registration order had to carry an ordinal beside the identifier. An
ordinal and an identifier that disagree is a failure nothing downstream can
detect or repair, and that turned out to cost more than the ordering it hid.

A later correction or dependency update cites the existing identifier and
becomes version 2, 3, and so on. Automated registration requires the same source
repository, selected project path, and Comparator configuration path as the
current version. A source commit already present in that identifier's version
history cannot be registered again. A repository transfer needs explicit
operator review. Earlier entry files and their source commits remain unchanged.

The existing identifier does not authorise an update. Every version separately
uses the checks in section 4: the submitter proves write access to the submitted
repository and declares that they are a responsible author or maintainer of the
substantive formalisation, or have approval from one. The declaration is the
authorisation basis; optional evidence may document approval, while repository
write access alone is deliberately not treated as authorship or approval.

### 9.1 Exceptional registry metadata corrections

In exceptional circumstances, a Palomar Technical Maintainer may correct
public descriptive metadata by appending a new, visibly labelled version. This
route is for registry mistakes or materially inaccurate metadata, not for
changing the formalization under review. It may change only the title,
abstract, authors, arXiv or MSC2020 classification, responsible maintainers,
mathematical sources, or related formalizations.

Such a correction must retain the exact repository, source commit, selected
project path, `formalization.yaml` path, and Comparator configuration path of
the current active version. It reuses that version's mechanical verification,
render, preservation, and trust evidence, and receives a new automated review
of the corrected effective metadata. The public record identifies the
inherited evidence, lists the exact fields changed, and gives a required public
plain-text explanation under the attribution “Palomar / Registry correction”.
The operator's account remains private.

A maintainer correction is an exceptional registry housekeeping act. It does
not state or imply that Palomar, its maintainers, or the project authors
approve, endorse, or take authorship of the formalization. Earlier versions
remain immutable and citable.

Technical Maintainers follow the
[Registry correction runbook](docs/maintainer-corrections.md), which identifies
the protected production entry point, the required checks, and the public
post-registration verification.
