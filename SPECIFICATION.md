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

The submission workflow publishes `mechanical-report.json` as a run artifact and
as a human-readable fenced JSON copy in the issue comment. The trusted workflow
artifact, not comment text, is the publication authority. A passing report binds:

- the issue and submitter;
- `owner/repo` and the resolved 40-character commit;
- hashes of `Challenge.lean`, `Solution.lean`, `formalization.yaml`, and
  `comparator.json`;
- the Lean toolchain;
- compared theorem and definition names;
- the full project dependency set, the transitive `Challenge.lean` source
  closure, its allowlist/Palomar provenance, exact indexed record versions and
  source hashes, and challenge size;
- pinned Comparator, lean4export, and landrun commits;
- the workflow run URL and timestamp.

The reviewer resolves the successful trusted workflow run, downloads its
issue-scoped artifact, checks that the workflow revision remains in trusted
history, and independently binds the artifact to the issue, source commit, and
run URL. A missing, stale, ambiguous, or malformed artifact fails closed.

## Review packet

The reviewer checks out source and policy into read-only local directories. Each
review pass receives:

1. the pass prompt at the recorded policy commit;
2. any binding policy documents named by that rubric step, including the
   editorial floor and score threshold for notability and synthesis;
3. `formalization.yaml`, `Challenge.lean`, `Solution.lean`,
   `comparator.json`, `lakefile.toml`, and `lean-toolchain`;
4. the issue metadata and mechanical report;
5. every exact source file from a Palomar-indexed dependency that occurs in the
   mechanically computed Challenge closure;
6. earlier pass results when the rubric says so.

Engines must emit JSON matching the per-pass shape in the rubric. Raw model
outputs, normalized results, and the final report are retained in the review
work directory. The synthesis output must validate against
`schemas/review.schema.json`.

Every submission-controlled input and every earlier model result is framed as
untrusted evidence, with the binding instruction repeated after the evidence.
Review engines run without write or shell tools, and their public prose is
rendered inertly. A model result is still advisory: applying a decision never
reruns the model and can post only the existing schema-validated `review.json`
whose issue, source, mechanical report, and policy commit an operator inspected.

`rubric.json` versions the engine contract. Version 2 declares score ownership
per pass, requires strict evidence findings, and binds synthesis decisions to
the evidence-pass scores. Reviewers must retain version 1 support for historical
policy commits and reject unknown rubric versions.

## Publication

On `accept`, the reviewer renders the accepted `Challenge.lean` and prepares
(but does not merge) a pull request to `PalomarDatabase`. Rendering is a
required publication step, but an infrastructure or renderer failure does not
reverse the editorial decision: publication remains pending and may be retried.
The record uses the submission issue for a new permanent ID.
For an update, it verifies the requested existing ID and chooses one more than
the greatest registered version. Database CI verifies the schema, filename, and
exact `index.json` projection.

The database record includes a required `challenge_render` object binding the
content-addressed bundle, entrypoint, Verso commit, renderer commit, Landrun
commit, and render time. The immutable bundle is retained under `renders/` in
the database repository. Merging the database PR is the publication event.
Closing or labeling the submission issue is a separate explicit operator
action.

## Security boundary

Submission source is hostile. No credential is present in the verification or
render execution environment.
The trusted reporting job has an issue-scoped token but never executes source or
source-derived shell text. Comparator performs Lean elaboration under landrun.
Mutable third-party action selectors are forbidden.

Rendering uses a trusted synthetic Lake workspace and an exact Verso revision
selected for the accepted Lean toolchain. It must not run a submitted Lakefile,
`lake update`, package post-update hooks, or source-derived commands outside
Landrun. Every Lake, Lean, Verso, and source-derived executable runs inside
Landrun without network access or credentials and with generous emergency
ceilings for time, memory, processes, files, and output. Resource exhaustion is
a retryable infrastructure result, never a mathematical rejection. Fixed-host cache downloads are performed by
trusted code outside source-derived execution and unpacked inside Landrun.

Published render HTML is sanitized, carries a restrictive CSP, and is served
from the PalomarDatabase GitHub Pages site. PalomarWeb treats the database as
the sole source of truth and embeds eligible Challenges in an iframe with
`sandbox="allow-scripts"` and no `allow-same-origin`. It always links to the
commit-pinned `Challenge.lean`; a Challenge is eligible for inline display only
when exactly one declaration is compared and the file is at most 100 lines and
32 KiB. Other entries link to a dedicated wrapper with the same sandbox.

Arbitrary pinned Git dependencies are permitted in the proof project. The
mechanical gate applies only to the transitive source closure of
`Challenge.lean`; imported sources must resolve to Lean core, the allowlisted
Mathlib/Tau Ceti closure, or an exact previously accepted Palomar record version.
For an indexed import, the verifier independently checks out its recorded
repository and commit, verifies its pinned nested manifest, rebuilds it with the
network closed, freezes the output ahead of candidate paths, and verifies every
imported source byte. An unindexed source reached recursively is still rejected.
This makes the import reproducible and reviewable; it does not promote the
earlier project to Mathlib-level trust, so the record remains `qualified`.
Dependencies reached only by `Solution.lean` remain unrestricted apart from the
ordinary full-commit and confinement requirements.

This prototype accepts only public GitHub repositories. Private-repository App
tokens, source retention, DOI minting, rate limiting, and automatic dependency
updates are intentionally deferred.
