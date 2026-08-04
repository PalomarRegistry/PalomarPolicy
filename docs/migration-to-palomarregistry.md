# Migrating Palomar to the PalomarRegistry organisation

Date: 2026-08-04. Settles the institutional-home half of `TODO.md` decision 8.

Palomar's repositories live under a personal account, `kim-em`. They move to the
[`PalomarRegistry`](https://github.com/PalomarRegistry) organisation. This
document is the checklist. It exists because a transfer is close to a one-shot
operation: GitHub's redirects are helpful but incomplete, and one of the ways
they break is permanent.

## What moves and what does not

Six repositories transfer:

`PalomarSubmission`, `PalomarDatabase`, `PalomarPolicy`, `PalomarReviewer`,
`PalomarTemplate`, `PalomarWeb`.

Three repositories referenced by the database **do not transfer**:
`kim-em/erdos-unit-distance-comparator`, `kim-em/PrimeNumberTheoremAnd`, and
`kim-em/leancert`. They are submitted work and cited sources, not registry
infrastructure. A registry that absorbs the artifacts it certifies has stopped
being a registry, and the fact that these happen to share an owner with the
registry today is an accident to be preserved, not tidied away.

## What GitHub does and does not redirect

- Repository, issue, comment, blob and raw URLs redirect after a transfer.
- **GitHub Pages URLs do not redirect.** `kim-em.github.io/PalomarDatabase/`
  simply stops resolving.
- **Recreating a repository at an old name destroys that name's redirects
  permanently.** The old `kim-em/Palomar*` names must therefore be reserved and
  never reused, for as long as any published record cites them. Three published
  records do.
- Actions run URLs have not been confirmed to redirect. Verify before relying on
  them; the mitigation is already in place either way, below.

## The publication freeze

Every schema from v2 to v6 pins the submission repository:

```json
"submission": { "repository": { "const": "kim-em/PalomarSubmission" } }
```

Published entries declare schema 2, 3 and 5, so `schema-v2`, `-v3` and `-v5` are
frozen and must never change. `schema-v4` and `schema-v6` are editable, because
no entry declares them.

The consequence is that after the transfer, no entry can be published until a
schema accepts the new repository, because a new record's
`submission.repository` would fail the `const` in every existing schema.

**Decision: accept the freeze.** No schema is edited to work around it.
Publication resumes with `schema-v7`, which has to restructure submission
identity anyway for submissions that arrive without a GitHub issue. Anyone
mid-submission is told to wait rather than being published against a widened
stopgap schema that would then be discarded.

## Before transferring

1. **Capture evidence for pre-v5 entries.** Done: `PalomarDatabase` PR #24. The
   v1 and v2 records cite only a `workflow_url` and a `report_url`, with no
   durable evidence bundle, so they were the records most exposed to a transfer.
   Their run artifacts and review comments are now archived in the database
   itself under `legacy-evidence/`, bound to the entries and frozen.
2. **Set organisation default member permissions.** These apply to repositories
   on arrival, so setting them afterwards means a window in which they are wrong.
3. **Decide the base member permission is `read`,** and grant write per
   repository, rather than inheriting a permissive default.

## The reference inventory

Every `kim-em` reference falls into exactly one of four classes. After the
migration an automated check asserts that every surviving reference is
explicitly allowlisted under one of the first three.

### Immutable, never change

| Path | Why |
| --- | --- |
| `PalomarDatabase/entries/**` | Published records are immutable byte for byte. |
| `PalomarDatabase/evidence/**` | Frozen with the record it belongs to. |
| `PalomarDatabase/legacy-evidence/**` | Frozen on capture. |
| `PalomarDatabase/schema-v{2,3,5}.json` | Frozen: published entries declare them. |
| `PalomarDatabase/schema-v{4,6}.json` | Editable, but deliberately left alone. |

These keep saying `kim-em` forever. That is correct: they are historical
statements about where a submission was verified in July and August 2026, and
they were true. Rewriting them would be indistinguishable from fabricating them.

### Submitter-owned, never change

`kim-em/erdos-unit-distance-comparator`, `kim-em/erdos-unit-distance`,
`kim-em/PrimeNumberTheoremAnd`, `kim-em/leancert`, wherever they appear,
including in `PalomarSubmission/.github/workflows/compatibility.yml` and
`PalomarPolicy/research/candidate-submissions.md`.

### Runtime validation, must accept both

Two places pin the submission repository as a constant and derive URL prefixes
from it, so simply repointing them would reject all three published records:

- `PalomarDatabase/tools/validate.py:590`
- `PalomarWeb/assets/security.mjs:11`

**Prescribed fix:** stop deriving expected URLs from one global constant. Take
the repository from the entry's own `submission.repository`, check that value
against a known set of submission repositories (the historical one and the
current one), and derive the run, issue and comment URL prefixes from it. A
record must be internally consistent; it need not name whichever repository the
validator was written in.

### Must change

| Path | Reference |
| --- | --- |
| `PalomarDatabase/tools/build_feeds.py:14-15` | web base, feed base |
| `PalomarDatabase/README.md`, `docs/append-only.md`, `docs/render-origin.md` | prose and links |
| `PalomarWeb/assets/app.js:25-27` | canonical web base, feed base, database source base |
| `PalomarWeb/assets/security.mjs:4-5` | default database raw URL, default render base |
| `PalomarWeb/{index,entry,render,404,about}.html` | CSP `frame-src`, RSS links, prose |
| `PalomarWeb/tests/{security.test.mjs,rendering.test.js,site.spec.js,fixture_server.py}` | fixtures follow the code |
| `PalomarReviewer/src/palomar_reviewer/cli.py` | submission, policy, database repositories |
| `PalomarReviewer/{README.md,.github/workflows/ci.yml,tests/test_cli.py}` | |
| `PalomarSubmission/README.md`, `docs/launch-security-review.md` | |
| `PalomarSubmission/.github/ISSUE_TEMPLATE/{config.yml,submit.yml}` | policy links |
| `PalomarSubmission/.github/workflows/{submission,compatibility}.yml` | database references |
| `PalomarSubmission/allowed-challenge-repositories.json` | database reference |
| `PalomarPolicy/README.md` | reviewer link |
| `PalomarTemplate/README.md` | CI badge, policy and submission links |
| `PalomarDatabase/tests/{conftest.py,test_validate.py}` | fixtures follow the code |

`kim-em.github.io` references move to `palomarregistry.github.io` first, and then
to `palomar-registry.org` when DNS is in place. Doing it in that order keeps a
migration failure distinguishable from a DNS failure.

## Not in the working tree

Easy to forget, because grep does not find them:

- GitHub Pages configuration on `PalomarDatabase` and `PalomarWeb`, including
  the custom domain and enforced HTTPS.
- Branch protection rules and rulesets on every repository.
- Actions secrets and variables.
- Installed GitHub Apps and their repository access.
- Repository topics, descriptions, and homepage URLs.
- Issue labels, in particular the `status:*` set `PalomarSubmission` relies on.
- Webhook subscriptions.
- The reserved old repository names.

## After transferring

1. Re-enable GitHub Pages on `PalomarDatabase` and `PalomarWeb`; the transfer
   does not carry the deployment.
2. Run every repository's suite, then `tools/validate.py`,
   `tools/check_append_only.py --history main`, and `tools/build_feeds.py`.
3. Load the site, resolve a feed, and confirm the render iframe still displays.
4. Independently test an old repository URL, an old issue URL, an old comment
   URL, an old raw URL, and an old Actions run URL. Record which redirect.
5. Run the allowlist check and confirm every surviving `kim-em` reference is
   accounted for by one of the three permitted classes above.
