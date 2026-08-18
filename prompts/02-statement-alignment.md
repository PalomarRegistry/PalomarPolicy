# Informal-to-Lean statement review

Determine whether every declaration selected by Comparator expresses the
mathematical claim presented in the available narrative. Assemble that account
from Challenge module documentation, selected declaration docstrings, the
project README, and `formalization.yaml`; it may be divided across locations
and need not be duplicated.

Separately assess `project.description` as the public description shown beneath
the registry title. For every Comparator-selected theorem and definition,
record whether that description points to it directly, covers it as part of a
clearly identified group of results, or misses it. The description need not
repeat a complete formal statement, declaration name, every hypothesis, or
every edge case, but it must let a mathematical reader identify the subject and
principal outcome. A project name, workflow account, proof-status claim,
editorial judgment, or generic claim to formalize something is not sufficient.
If any selected declaration is missing, create a concrete finding and fail this
pass. Treat a materially false or misleading description as an alignment
failure as well.

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
  "verdict": "pass|warn|fail",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"statement_alignment": 4},
  "trust_level": null,
  "sources_checked": ["challenge_source", "repository@commit:path"],
  "declarations_checked": ["every Comparator theorem, then every definition, in order"],
  "description_coverage": [
    {"declaration": "each Comparator declaration in the same order", "coverage": "direct|collective|missing", "reason": "brief concrete reason"}
  ],
  "codes_checked": [],
  "internal_notes": [{"evidence": "prose and Lean locations", "message": "private audit note"}]
}
