# Metadata and provenance review

Assess the accuracy and sufficiency of the structured metadata and the
narrative account. Mechanical completeness is handled elsewhere; read the
values rather than rewarding populated fields or polished prose.

Required structured facts about provenance, sources, licence, classification,
authorship, automation, review status, any thin-wrapper relationship, scope, and known gaps
belong in `formalization.yaml`. Narrative explanation of the mathematics and
its development may instead live in Challenge module documentation, declaration
docstrings, or a selected-project README. Do not require duplication. A pointer
to a pinned in-repository document is acceptable only after resolving it and
recording `repository@commit:path` in `sources_checked`.

Project-level files may describe the whole repository, including results checked
by other Comparator configurations. Do not treat that as an overclaim about the
selected submission. Create a scope finding only when submission-specific prose
misstates what the selected Comparator configuration contains.

Check public claims against the Challenge source, mechanical report, submission
authorization, and repository documentation. Pay particular attention to
materially overstated scope, novelty or priority; contradictory repository
roles or maintainership; concealed limitations; inaccurate source or prior-
formalization claims; and ambiguous claims of human or automated review.
Automation metadata should identify material AI involvement and direct readers
to any fuller pinned account. Do not demand unavailable cost, hardware, prompt,
or timing detail merely for completeness. Review metadata should state the
actual level and basis of checking; do not demand a separate review that did
not occur.

Keep contribution roles distinct: bibliographic authorship, mathematical
discovery, formalization, verification, and communication may have different
credits. Palomar nevertheless reserves `project.authors` and
`project.responsible_maintainers` (including their compatibility aliases) for
humans. If an AI model, automated agent, system, session, or tool is named as an
author or responsible maintainer, create an error finding and use a `failure`
outcome. The correction must remove it from those identity fields, which must
name only the responsible human authors or maintainers, while retaining
accurate AI credit in `automation.methods` and the narrative contribution
account. Do not infer a violation merely from disclosed AI use, AI credit in
another contribution role, or an automated system named as a reviewer or source
author. Otherwise, report an attribution contradiction only when the same role
is attributed incompatibly or the roles are materially unclear.

For a thin wrapper, check that the substantive formalization is pinned at an
immutable revision. Responsible maintainers may describe the submitted wrapper
while authorization concerns the underlying project; that difference is
expected and is not itself a contradiction. Do not demand a prior-work citation
for a result credibly recorded as first presented by the formalization, but do
assess whether that originality claim and its literature context are accurate.

Optional detail, wording preferences, harmless empty optional fields, and
narrative facts already supplied by a verified pinned document belong in
`internal_notes`, not findings. Create findings only under the binding
materiality policy, and do not repeat an earlier finding.

Record every evidence location actually inspected in `sources_checked`. Use
empty `codes_checked` and `declarations_checked` lists. Set only `clarity` and
`provenance`; all other scores are null in the enforced output schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "metadata",
  "outcome": "neutral|warning|failure",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"clarity": 4, "provenance": 4},
  "trust_level": null,
  "sources_checked": ["formalization_metadata", "repository@commit:path"],
  "declarations_checked": [],
  "codes_checked": [],
  "internal_notes": [{"evidence": "metadata field or pinned document", "message": "private audit note"}]
}
