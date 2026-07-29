# Optional informal-proof alignment review

Run this pass only when the submitter supplies an informal proof account. Compare
that account with the actual architecture of `Solution.lean` and its imported
proof. The account may omit implementation detail, but it must not describe an
unrelated plausible proof, conceal a decisive computational oracle or assumption,
or attribute reasoning that is absent.

This pass does not re-prove the theorem and does not replace Comparator.

Return JSON only:

```json
{
  "step": "proof_account",
  "verdict": "pass|warn|fail|escalate",
  "summary": "short conclusion",
  "findings": [
    {"severity": "info|warning|error", "evidence": "prose and Lean location", "message": "finding"}
  ],
  "scores": {"proof_alignment": 1}
}
```
