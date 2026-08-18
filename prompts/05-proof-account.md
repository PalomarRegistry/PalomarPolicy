# Optional informal-proof alignment review

Run this pass only when the submission contains an informal proof account.
Locate all such passages in module documentation, declaration docstrings, the
README, and metadata, and compare them with the actual architecture of the
recorded Solution source and its imported proof.

The account may omit implementation detail. Create a finding only when it
materially misstates the proof, conceals a decisive assumption or computational
component, or describes an unrelated argument. Distinguish a harmless omitted
detail from a missing decisive step. Removal of an inaccurate optional proof
account is a valid correction. Record accurate correspondences and ordinary
implementation omissions in `internal_notes`; do not create public praise or
repeat an earlier finding.

This pass does not re-prove the theorem and does not replace Comparator. Audit
all selected declarations rather than one headline. List every Comparator
theorem followed by every Comparator definition in configuration order in
`declarations_checked`. Record every prose, Solution, and imported-proof
location inspected in `sources_checked`; use an empty `codes_checked` list.
Set only `proof_alignment`; all other scores are null in the enforced output
schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "proof_account",
  "outcome": "neutral|warning|failure",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"proof_alignment": 4},
  "trust_level": null,
  "sources_checked": ["solution_source", "repository@commit:path"],
  "declarations_checked": ["every Comparator theorem, then every definition, in order"],
  "codes_checked": [],
  "internal_notes": [{"evidence": "prose and proof locations", "message": "private audit note"}]
}
