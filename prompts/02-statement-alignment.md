# Informal-to-Lean statement review

Determine whether every declaration selected by Comparator expresses the
mathematical claim presented in the available narrative. Assemble that account
from Challenge module documentation, selected declaration docstrings, the
project README, and `formalization.yaml`; it may be divided across locations
and need not be duplicated.

Treat `project.description` as a concise, project-wide public abstract, not a
declaration manifest or a description limited to this Comparator selection. It
may accurately describe additional project results, including results checked
by another configuration. Use the selected configuration as evidence that the
abstract at least points, directly or collectively, to the mathematical subject
and principal result families represented by this selection. It need not name
every theorem, variant, supporting declaration, definition, hypothesis,
constant, or edge case. Create a finding only when the abstract is materially
false or misleading, unrelated to the selected work, or omits a distinct
principal result family represented by the selection. Do not create a finding
merely because an individual declaration is not separately identifiable.
Missing mechanically required metadata is handled by intake validation, not
editorial review.

For each selected declaration, compare the concrete prose and Lean locations.
Check definitions, quantifiers, hypotheses, coercions, degenerate cases, and
claimed scope. Comparator proves that Solution discharges Challenge; it does
not establish that Challenge says what the prose advertises. Apply the binding
effective-domain rule rather than reporting every totalized or degenerate
operation syntactically present.

Create a finding when the statement can be vacuous or materially weaker,
stronger, or different from the presented claim, when a headline claim lacks
any assessable narrative, or when contradictory prose leaves the indexed claim
unclear. State the concrete semantic consequence and whether Lean, prose, or
both must change. Put exact agreement checks and excluded edge cases in
`internal_notes`. Do not repeat a concern already established in
`previous_findings`.

List every Comparator theorem followed by every Comparator definition in
configuration order in `declarations_checked`. Record every prose and Lean
location inspected in `sources_checked`; use an empty `codes_checked` list.
Set only `statement_alignment`; all other scores are null in the enforced
output schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "statement_alignment",
  "outcome": "neutral|warning|failure",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"statement_alignment": 4},
  "trust_level": null,
  "sources_checked": ["challenge_source", "repository@commit:path"],
  "declarations_checked": ["every Comparator theorem, then every definition, in order"],
  "codes_checked": [],
  "internal_notes": [{"evidence": "prose and Lean locations", "message": "private audit note"}]
}
