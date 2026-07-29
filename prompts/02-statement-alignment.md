# Informal-to-Lean statement review

Determine whether each theorem named in `comparator.json` actually expresses the
corresponding informal claim in `formalization.yaml`. Read quantifiers,
hypotheses, definitions, namespaces, coercions, degenerate cases, and claimed
scope closely. Check that the metadata does not silently generalize beyond or
market a weaker surrogate for the Lean statement.

Comparator establishes that Solution proves Challenge; it does not establish
that Challenge formalizes the advertised mathematics. That semantic question is
your task. Cite concrete Lean fragments and concrete prose. Do not accept on name
similarity.

Return JSON only:

```json
{
  "step": "statement_alignment",
  "verdict": "pass|warn|fail|escalate",
  "summary": "short conclusion",
  "findings": [
    {"severity": "info|warning|error", "evidence": "file/declaration", "message": "finding"}
  ],
  "scores": {"statement_alignment": 1}
}
```

Scores are integers 1–5. A mismatch affecting the headline claim is `fail`.
