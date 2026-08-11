# Subject-classification review

Check every submitted arXiv subject class and MSC2020 code against the actual
mathematical result in the Challenge source and metadata. Intake has already
checked that the identifiers exist; use the binding classification guide for
their meanings.

This is a plausibility screen, not an optimization exercise. A code passes when
it has a reasonable substantive connection to the result. Do not create a
finding because another code would be more specific, conventional, or useful,
or because a defensible code is somewhat broad. Record that judgment in
`internal_notes`. Create a finding only for a materially unrelated,
misleading, or purely tooling-based classification, or when the evidence does
not identify enough mathematics to assess the code responsibly.

Record every checked code in `codes_checked`, first all arXiv codes and then
all MSC2020 codes, preserving metadata order and using `arxiv:CODE` and
`msc2020:CODE`. Record the evidence files in `sources_checked`; use an empty
`declarations_checked` list.

Use `findings: []` and `verdict: pass` when there is no material criticism.
Keep positive topical reasoning and harmless classification alternatives in
`internal_notes`. Scores are integers 1–5; set only `classification` and set
every other score to null in the enforced output schema.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "classification",
  "verdict": "pass|warn|fail",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"classification": 4},
  "trust_level": null,
  "sources_checked": ["formalization_metadata", "challenge_source"],
  "declarations_checked": [],
  "codes_checked": ["arxiv:CODE", "msc2020:CODE"],
  "internal_notes": [{"evidence": "code and mathematical claim", "message": "private audit note"}]
}
