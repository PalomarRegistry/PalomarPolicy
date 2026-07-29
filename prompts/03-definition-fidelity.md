# Definition and trust-surface review

Audit every nontrivial definition, structure, instance, notation, and imported
project-specific concept on which a compared statement depends. Ask whether each
matches the ordinary mathematical meaning asserted by its docstring and metadata.
Look especially for definitions that bake in the desired conclusion, omit
well-formedness conditions, collapse degenerate cases, or make a definition hole
vacuous.

Use the mechanically computed transitive challenge-source closure and challenge
size to assess auditability. Dependencies used only by `Solution.lean` are not
part of the statement trust surface and must not be penalized. Mathlib-only
challenge imports support a `high` trust level. Tau Ceti or Palomar-indexed
challenge imports may still pass, but must be identified as a qualified trust
surface with useful warnings.

Return JSON only:

```json
{
  "step": "definition_fidelity",
  "verdict": "pass|warn|fail|escalate",
  "summary": "short conclusion",
  "findings": [
    {"severity": "info|warning|error", "evidence": "file/declaration/import", "message": "finding"}
  ],
  "scores": {"definition_fidelity": 1, "auditability": 1},
  "trust_level": "high|qualified"
}
```

Scores are integers 1–5. A manufactured or materially misleading definition is
`fail`.
