# Literature and editorial-floor review

Assess the actual selected mathematics as a selective research editor. Formal
verification, effort, length, and polish do not by themselves establish
research interest. Novelty is not required.

For each distinct selected result group, ask whether it could plausibly warrant
a research paper or serious note and whether a credible research area and
plausible research audience can be identified. Related corollaries may be
grouped. A niche result may pass. If either requirement is not affirmatively
established, notability is below the mandatory floor, the pass fails, and the
finding should describe the evidentiary limit rather than claim more than the
evidence supports.

Check the structured source and related-formalization facts, the narrative
literature account, important citations, and obvious prior work. Search
independently when browsing is available and record the sources used. Do not
infer novelty from a missing citation or penalize an original result merely for
lacking a prior mathematical source. Informal communication, correspondence,
and folklore may be disclosed honestly even when independently unverifiable;
it cannot establish novelty, priority, or reception, but its medium is not
itself a defect.

Create findings for material misattribution, unsupported priority or novelty,
an omitted prior result or formalization that changes the public account, or
failure of the research-interest floor. Put minor bibliographic slips,
additional plausible comparisons, positive evidence, and unsuccessful search
paths in `internal_notes`. Do not manufacture a negative public comment merely
to demonstrate critical thought, and do not repeat an earlier finding.

List every Comparator theorem followed by every Comparator definition in
configuration order in `declarations_checked`. Record repository evidence and
external URLs actually inspected in `sources_checked`; use an empty
`codes_checked` list. Set only `notability` and `literature`; all other scores
are null in the enforced output schema.

Use the binding notability anchors in `CONTRIBUTING.md`. A notability score
below the rubric minimum must use `fail`; better prose alone is not a proposed
fix when the mathematical result itself does not clear the floor.

Return one bare JSON object and nothing else: no code fence, no surrounding prose.

{
  "step": "literature_notability",
  "verdict": "pass|warn|fail",
  "summary": "short conclusion",
  "findings": [],
  "scores": {"notability": 4, "literature": 4},
  "trust_level": null,
  "sources_checked": ["repository@commit:path", "https://example.invalid/source"],
  "declarations_checked": ["every Comparator theorem, then every definition, in order"],
  "codes_checked": [],
  "internal_notes": [{"evidence": "literature source or selected result", "message": "private audit note"}]
}
