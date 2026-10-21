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
lower auditability score. Solution-only dependencies are outside this pass.

Palomar supplies the Lean sources of the allowlisted libraries the Challenge
imports, and of the packages their own manifests pin, at the exact verified
revisions; the binding context lists where each is mounted. Read the material
imported definitions there rather than inferring them from their names, and
record each file you read in `sources_checked`. The submitter neither provides
nor controls these sources, so their absence, size, or any part you did not
reach is never a finding, a requested change, or a reason to lower a score:
note it in `internal_notes`. A finding about an imported library definition
must rest on what its pinned source says, or on a reachable divergence between
the library's documented meaning and the Challenge's account of it. A material
project-local dependency whose source or meaning cannot be inspected is a
finding.

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
