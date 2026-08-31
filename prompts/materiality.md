# Binding materiality and reporting policy

A `finding` is an author-facing criticism. It is shown to the submitter and,
if they register the review, published permanently. Create one only when the
evidence supports at least one of these conclusions:

- the compared Lean statement may be vacuous, materially weaker, materially
  stronger, or materially different from the mathematical claim presented;
- a public claim about scope, provenance, priority, authorship, licence,
  automation, or review is materially unsupported, misleading, or
  contradictory;
- a subject classification is egregiously off-topic for both the result and any
  plausible proof;
- the selected statements or their trusted dependencies cannot be audited from
  the pinned evidence; or
- the mathematical result does not clear Palomar's research-interest floor.

Do not create findings for terminology preferences, stylistic duplication,
optional detail, a plausible but non-optimal classification, or behaviour that
the stated hypotheses provably exclude. Do not demand that narrative prose be
copied into structured metadata when the structured facts are present and a
pinned in-repository document supplies the narrative evidence. A pointer is
evidence only after it has actually been resolved; record the immutable
`repository@commit:path` in `sources_checked`. An external document cannot
replace required structured metadata.

Do not create editorial findings for missing or malformed fields whose presence
and shape are hard mechanical intake requirements; validation and repair own
those defects. For legacy evidence predating such a requirement, assess the
available substantive account and record the structural absence only in
`internal_notes`.

For classifications, presume possible proof relevance without checking it.
Only an egregiously off-topic code can support a finding.

Project-level files may describe the whole repository, including results checked
by other Comparator configurations. Do not treat that as an overclaim about the
selected submission. Create a scope finding only when submission-specific prose
misstates what the selected Comparator configuration contains.

Interpret claims such as “unconditional” or “requires no hypotheses” in their
mathematical context. Domain restrictions, typing conditions, and conditions
defining the objects under study are not omitted hypotheses; create a finding
only when the prose conceals a substantive mathematical assumption that
materially narrows the claimed result.

Keep contribution roles distinct: bibliographic authorship, mathematical
discovery, formalization, verification, and communication may have different
credits. Automated systems may be credited for those contribution roles, but
Palomar reserves `project.authors` and `project.responsible_maintainers` for
humans who can hold credit and responsibility. Naming an AI model, automated
agent, system, session, or tool in either identity field is a material finding;
accurately disclosing the same system in automation metadata, narrative credit,
source attribution, or review metadata is not. Otherwise, report a
contradiction only when the same role is attributed incompatibly or the roles
are materially unclear.

An inspectable cited source may supply literature context; do not require that
context to be duplicated in repository prose.

Reviewer-side inability to access a precisely cited source is not an
author-facing defect. Record the limitation in `internal_notes`; absent
affirmative evidence of a material omission or unsupported claim, make no
finding.

Lean operations may be total outside their ordinary mathematical domain: for
example division at zero, truncated subtraction, extrema of empty sets,
unbounded suprema or infima, nonconvergent infinite operations, and defaulted
extraction. Assess such behaviour under the theorem's actual hypotheses. If
the hypotheses exclude it, record the exact exclusion argument in
`internal_notes` and make no finding. If they do not, explain in a finding how
the statement can become vacuous or materially change and what must be fixed.
If exclusion cannot be established from the pinned evidence, do not silently
suppress the concern.

An informal account may rely on immediate consequences of definitions it
cites. Require an extra hypothesis or conjunct only when the informal claim
asserts something the compared statement does not entail.

Use `internal_notes` for private proof of work: positive checks, harmless edge
cases, excluded failure modes, and concerns considered but found non-material.
Each note must cite concrete evidence. These notes are retained for operator
audit but are not shown to the submitter or published. They cannot justify a
warning, requested change, revision, or rejection unless the concern is
promoted to a finding.

The check `summary` may also be published. Keep it to the overall conclusion and
the material findings, if any; do not use it to surface private notes or
non-material criticism indirectly.

Use `findings: []` for a clean check. A `neutral` outcome has no findings; a
`warning` has at least one warning finding; and a `failure` has at least one
error finding. Combine
declarations only where one root cause and one correction cover all of them.
Do not repeat a material criticism already present in `previous_findings`;
continue the independent audit, record any additional private reasoning in
`internal_notes`, and leave the public finding with the check that first
established it.

Every public message must state the concrete consequence and, where the defect
is correctable, the correction. It must not mention or imply a private score.
Assess the work and its presentation, never the submitter.
