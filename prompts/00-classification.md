# Subject-classification review

Review every arXiv and MSC2020 code using the binding guide; intake has already
checked that each identifier exists.

This is only an egregious-mismatch screen. Presume author-selected codes are
legitimate. Accept a code if it could describe the statement, an ancillary
result, or mathematics in the proof; do not inspect the proof or require
evidence. Reject only a code unmistakably off-topic for the result and any
plausible proof. Absence from the Challenge or metadata, breadth,
unconventionality, and better alternatives are not criticisms. Lean or AI use
alone is not mathematical relevance. If a proof connection can be imagined,
use `findings: []` and do not lower the score.

Record every checked code in `codes_checked`, first all arXiv codes and then
all MSC2020 codes, preserving metadata order and using `arxiv:CODE` and
`msc2020:CODE`. Record the evidence files in `sources_checked`; use an empty
`declarations_checked` list.

Use `findings: []` and `outcome: neutral` when there is no material criticism.
Keep positive topical reasoning and harmless classification alternatives in
`internal_notes`. Scores are integers 1–5; set only `classification` and set
every other score to null in the enforced output schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "classification",
  "outcome": "neutral|warning|failure",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"classification": 4},
  "trust_level": null,
  "sources_checked": ["formalization_metadata", "challenge_source"],
  "declarations_checked": [],
  "codes_checked": ["arxiv:CODE", "msc2020:CODE"],
  "internal_notes": [{"evidence": "code and mathematical claim", "message": "private audit note"}]
}
