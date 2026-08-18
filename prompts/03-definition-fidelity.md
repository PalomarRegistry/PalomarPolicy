# Definition and trust review

Audit the nontrivial definitions, structures, instances, notation, and imported
project-specific concepts on which each selected statement materially depends.
Compare their actual Lean definitions with the mathematical meaning claimed in
module documentation, docstrings, the README, and metadata.

Look for definitions that manufacture the conclusion, omit a necessary
well-formedness condition, collapse a reachable case, or otherwise make the
claim vacuous or materially different. Apply the binding effective-domain rule:
inspect totalized and defaulted behaviour under the theorem hypotheses, record
a proved exclusion in `internal_notes`, and create a finding only when the bad
case remains reachable or cannot be audited from the pinned evidence.

Use the mechanically computed Challenge dependency closure and size to assess
auditability. Mathlib-only Challenge imports give `high` trust. Allowed Tau
Ceti imports give `qualified` trust but are not by themselves a criticism or a
lower auditability score. A material dependency whose source or meaning cannot
be inspected is a finding. Solution-only dependencies are outside this pass.

Record concrete positive inspections and harmless limitations in
`internal_notes`. Do not create public praise and do not repeat an earlier
material finding. A manufactured or materially misleading definition is a
failure.

List every Comparator theorem followed by every Comparator definition in
configuration order in `declarations_checked`. Record actual files, imports,
and resolved pinned documents in `sources_checked`; use an empty
`codes_checked` list. Set only `definition_fidelity` and `auditability`, and set
`trust_level` to `high` or `qualified`; all other scores are null in the
enforced output schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "definition_fidelity",
  "outcome": "neutral|warning|failure",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"definition_fidelity": 4, "auditability": 4},
  "trust_level": "high|qualified",
  "sources_checked": ["challenge_source", "repository@commit:path"],
  "declarations_checked": ["every Comparator theorem, then every definition, in order"],
  "codes_checked": [],
  "internal_notes": [{"evidence": "definition and hypotheses", "message": "private audit note"}]
}
