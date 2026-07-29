# Palomar protocol v1

## Separation of responsibilities

| Repository | Responsibility | Must not do |
| --- | --- | --- |
| `PalomarDatabase` | Append-only accepted records and schemas | Intake, execute Lean, review, render UI |
| `PalomarSubmission` | Issue intake and mechanical verification | Run AI reviewers, decide editorial policy |
| `PalomarPolicy` | Style guide, rubric, prompts, review schema | Hold accepted records or credentials |
| `PalomarReviewer` | Operator-run AI review and database-PR preparation | Run in submission CI or merge its own PR |
| `PalomarWeb` | Read-only human presentation | Become a second source of registry truth |

## Submission state machine

```text
issue opened
  -> status:verifying
  -> status:awaiting-review | status:changes-requested
  -> status:review-in-progress
  -> status:accepted | status:changes-requested | status:rejected | status:escalated
```

Exactly one `status:*` label should be present. The mechanical report comment is
identified by the marker `<!-- palomar-mechanical-report -->`; review reports use
`<!-- palomar-editorial-review -->`.

## Mechanical result

The submission workflow publishes `mechanical-report.json` as an artifact and
as a fenced JSON block in the issue comment. A passing report binds:

- the issue and submitter;
- `owner/repo` and the resolved 40-character commit;
- hashes of `Challenge.lean`, `Solution.lean`, `formalization.yaml`, and
  `comparator.json`;
- the Lean toolchain;
- compared theorem and definition names;
- the full project dependency set, the transitive `Challenge.lean` source
  closure, its allowlist/Palomar provenance, and challenge size;
- pinned Comparator, lean4export, and landrun commits;
- the workflow run URL and timestamp.

The report is evidence, not an authorization token. The reviewer independently
checks that the issue, commit, workflow conclusion, and report agree.

## Review packet

The reviewer checks out source and policy into read-only local directories. Each
review pass receives:

1. the pass prompt at the recorded policy commit;
2. `formalization.yaml`, `Challenge.lean`, `Solution.lean`,
   `comparator.json`, `lakefile.toml`, and `lean-toolchain`;
3. the issue metadata and mechanical report;
4. earlier pass results when the rubric says so.

Engines must emit JSON matching the per-pass shape in the rubric. Raw model
outputs, normalized results, and the final report are retained in the review
work directory. The synthesis output must validate against
`schemas/review.schema.json`.

## Publication

On `accept`, the reviewer prepares (but does not merge) a pull request to
`PalomarDatabase`. The record uses the submission issue for a new permanent ID.
For an update, it verifies the requested existing ID and chooses one more than
the greatest registered version. Database CI verifies the schema, filename, and
exact `index.json` projection.

Merging the database PR is the publication event. Closing or labeling the
submission issue is a separate explicit operator action.

## Security boundary

Submission source is hostile. No credential is present in the verification job.
The trusted reporting job has an issue-scoped token but never executes source or
source-derived shell text. Comparator performs Lean elaboration under landrun.
Mutable third-party action selectors are forbidden.

Arbitrary pinned Git dependencies are permitted in the proof project. The
mechanical gate applies only to the transitive source closure of
`Challenge.lean`; imported sources must resolve to Lean core, the allowlisted
Mathlib/Tau Ceti closure, or a commit already recorded in Palomar. This is a
source-provenance restriction, not a package-name or direct-lakefile heuristic.

This prototype accepts only public GitHub repositories. Private-repository App
tokens, source retention, DOI minting, rate limiting, and automatic dependency
updates are intentionally deferred.
