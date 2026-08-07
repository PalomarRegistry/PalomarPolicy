# Palomar infrastructure

Last reconciled against the live services on 2026-08-07.

This is the durable record of where Palomar runs, so that changing a host or
credential is a checklist rather than an excavation. The private canonical
database and the public data service are deliberately different systems; never
restore a direct raw-GitHub or GitHub Pages fallback for database data.

## Accounts and ownership

| Thing | Where | Notes |
| --- | --- | --- |
| Repositories | GitHub organization [`PalomarRegistry`](https://github.com/PalomarRegistry) | Moved from the `kim-em` personal account on 2026-08-04. Base member permission is `read`. `PalomarDatabase` and `PalomarSubmissionState` are private. |
| Preserved source | GitHub organization [`PalomarArchive`](https://github.com/PalomarArchive) | Public native forks and immutable record-specific tags. Base member permission is `write`; repository deletion, visibility changes, and private-repository creation are disabled. |
| Archive identity | GitHub user [`PalomarArchivist`](https://github.com/PalomarArchivist) | Dedicated 2FA-protected ordinary member of `PalomarArchive`, never an organization owner or a member of `PalomarRegistry`. |
| Domains | `palomar-registry.org` and `palomarregistry.org`, both at Cloudflare Registrar | Registrar and DNS are in the same Cloudflare account. |
| DNS, Workers, and R2 | Cloudflare account `d789bf36d237e0cb313be59b927c82bd` | Zones `f05ebb1809990a5d27e6d6a7d0d1ae85` for `palomar-registry.org` and `feea63b2ced3571a5ab5ce4ba516067f` for `palomarregistry.org`; nameservers `joyce`/`matias.ns.cloudflare.com`. |
| Website hosting | GitHub Pages, repository `PalomarWeb` | The website is static; its registry content is fetched at runtime from the public data Worker. |
| Public registry storage | Private R2 bucket `palomar-public-data` | Contains generated, active-only releases. The bucket itself is not public. |

The old `kim-em/Palomar*` repository names must stay reserved forever.
Recreating a repository at an old name destroys that name's GitHub redirect.
Published records contain immutable URLs, so the cost of losing a redirect only
grows.

## Source-preservation organization

`PalomarArchive` is a separate security boundary from the registry repositories.
Its [member privileges](https://github.com/organizations/PalomarArchive/settings/member_privileges)
and [authentication security](https://github.com/organizations/PalomarArchive/settings/security)
must remain configured as follows:

| Setting | Required value | Reason |
| --- | --- | --- |
| Base repository permission | Write | `PalomarArchivist` must add new preservation tags after its temporary per-fork administrator grant is removed. |
| Public repository creation | Allowed | GitHub creates a native organization fork as a new public repository. |
| Private repository creation | Disallowed | Accepted source is public and the archive must not become a private-data store. |
| Repository deletion | Disallowed for members | The archive identity cannot erase a preserved fork. |
| Repository visibility changes | Disallowed for members | The archive identity cannot hide or privatize a preserved fork. |
| Require two-factor authentication | Enabled | A member without 2FA cannot retain organization access. |

`PalomarArchivist` is an ordinary active member, not an owner. Creating a native
fork initially gives the creator direct administrator access to that repository.
Registration creates and reads back an immutable-tag ruleset, removes that
direct administrator grant, and only then writes the preservation tag using the
organization's base Write permission. The ruleset allows creation of new
`refs/tags/palomar/**/*` refs but has no bypass actor and rejects updates or
deletions of an existing preservation ref. Organizations cannot star GitHub
repositories, so preservation deliberately does not attempt to star sources.

## Service topology

| Host | Serves | Hosted by | Status |
| --- | --- | --- | --- |
| `palomar-registry.org` | Human-facing website | GitHub Pages, `PalomarWeb` | Live; HTTPS enforced |
| `www.palomar-registry.org` | Redirect to the apex | Cloudflare Redirect Rule | Live; 301; path and query preserved |
| `data.palomar-registry.org` | Filtered index, active entry records, schemas, renders, evidence, feeds, tombstones, and source availability | Cloudflare Worker `palomar-data` over private R2 | Live; read-only |
| `submit.palomar-registry.org` | Submission server | Cloudflare Worker `palomar-server` | Live |
| `palomarregistry.org` and `www.palomarregistry.org` | Defensive-domain redirects | Cloudflare Worker `palomar-domain-redirect` | Live; 308; path and query preserved |
| `palomarregistry.github.io/PalomarWeb/` | Legacy website location | GitHub Pages | Redirects to `palomar-registry.org` |
| `palomarregistry.github.io/PalomarDatabase/` | Nothing | No Pages site | Returns 404 |

The data path is:

```text
private PalomarDatabase main
  -> validate and build active-only snapshot
  -> remove private-only fields, including review scores
  -> upload immutable release objects to private R2
  -> verify every uploaded digest
  -> atomically update R2 _current.json
  -> palomar-data Worker exposes only allowlisted public paths
  -> PalomarWeb fetches https://data.palomar-registry.org/index.json
```

The accepted-source path is:

```text
author consents to register an accepted review
  -> PalomarReviewer verifies PALOMAR_ARCHIVE_TOKEN is PalomarArchivist
  -> resolve the submitted repository, every pinned Git dependency, and any
     separately recorded substantive formalization
  -> create or reuse one native PalomarArchive fork per GitHub fork network
  -> install and verify the immutable preservation-tag ruleset
  -> remove PalomarArchivist's direct repository administrator grant
  -> create and read back a record-specific tag for every accepted commit
  -> bind source-archive.json and the preservation map into the database record
  -> only then publish the PalomarDatabase registration branch
```

Any failure in that path stops registration before a database branch is
published. A source shape that a native fork cannot preserve is rejected during
mechanical verification rather than discovered after acceptance.

The Worker never exposes `_current.json`, release manifests, bucket listings,
the private `takedowns.json`, the complete canonical `index.json`, submitter
identity, or editorial review scores. These fields remain in the canonical
record but are deliberately absent from the generated public projection. It
has an R2 binding named `DATA` and no secrets. A missing or invalid
current-release pointer fails closed with 503; there is no raw-GitHub fallback.

`palomarregistry.org`, without the hyphen, is a defensive registration. It is a
separate Cloudflare zone and serves no content of its own.

## Cloudflare resources

| Resource | Configuration source | Deployment |
| --- | --- | --- |
| Worker `palomar-data-staging` | `PalomarDatabase/worker/wrangler.jsonc`, top level | Manual staging deployment at its `workers.dev` URL |
| Worker `palomar-data` | `PalomarDatabase/worker/wrangler.jsonc`, environment `production` | Manual deployment with `npx wrangler deploy --env production`; owns the `data` custom domain |
| R2 bucket `palomar-public-data` | `PalomarDatabase` publication tooling and the Workers `DATA` binding | Snapshots are published by private GitHub Actions, not by Worker deployment |
| Worker `palomar-server` | `PalomarServer/wrangler.jsonc` | Tested, uploaded, and promoted automatically on pushes to `main` |
| Worker `palomar-domain-redirect` | `PalomarServer/redirect/wrangler.jsonc` | Manual `npm run deploy:redirect` |
| `www.palomar-registry.org` redirect | Cloudflare zone Redirect Rule | Managed separately from Wrangler |

Deploying `palomar-data` changes the reader, not the data. Publishing a database
snapshot changes R2, not the Worker. Keep those operations separate so either
can be rolled back without changing the other.

## DNS

| Name | Configuration | Proxy/owner |
| --- | --- | --- |
| `palomar-registry.org` | GitHub Pages A records `185.199.108.153` through `185.199.111.153` and the corresponding AAAA set | DNS only; GitHub terminates TLS |
| `www.palomar-registry.org` | Placeholder record used only to enter Cloudflare's proxy | Proxied; Redirect Rule answers at the edge |
| `data.palomar-registry.org` | Worker custom domain declared by `PalomarDatabase/worker/wrangler.jsonc` | Managed by Cloudflare/Wrangler; do not recreate the retired GitHub Pages CNAME |
| `submit.palomar-registry.org` | Proxied record plus the Worker route in `PalomarServer/wrangler.jsonc` | Cloudflare Worker route |
| `palomarregistry.org`, `www.palomarregistry.org` | Worker custom domains declared by `PalomarServer/redirect/wrangler.jsonc` | Managed by Cloudflare/Wrangler |
| `_github-pages-challenge-palomarregistry` | GitHub organization Pages verification token | DNS only |

The apex remains grey-clouded because GitHub must receive the request and
terminate TLS. `data` must not point at `palomarregistry.github.io`: the old
CNAME was deleted before the Worker custom domain was deployed. Wrangler owns
the data custom-domain record, so changing it by hand risks disconnecting the
Worker.

## Render-origin isolation

Render bundles are untrusted output derived from submitter-controlled Lean
source. They are served from `data.palomar-registry.org`, while the parent site
is served from `palomar-registry.org`. PalomarWeb pins that render base and its
`frame-src` CSP to the data host. Embedded documents also use
`sandbox="allow-scripts"` without `allow-same-origin`, giving the frame an opaque
origin.

The distinct hostname, iframe sandbox, exact render CSP, database content
validator, trusted runtime hashes, and browser sanitizer are independent
layers. Do not remove one because the others exist. The data host must not set
application cookies or host the website, and render paths must remain immutable
and content-addressed.

## Certificates and HTTPS

GitHub Pages terminates TLS for `palomar-registry.org`; Cloudflare terminates TLS
for `www`, `data`, `submit`, and the defensive-domain redirects. Certificates
are managed by those providers and are not installed manually.

HTTPS enforcement for the website is the `https_enforced` setting on the
`PalomarWeb` Pages site. It does not apply to the data Worker. When testing a
Pages enforcement change, request an uncached path because Pages responses may
remain cached for several minutes:

```sh
curl -sSI "http://palomar-registry.org/nonexistent-$RANDOM.html" # expect 301
```

The hyphenated `www` Redirect Rule is:

```text
expression:  http.host eq "www.palomar-registry.org"
target:      concat("https://palomar-registry.org", http.request.uri.path)
status:      301, preserve_query_string: true
```

Create a redirect rule before proxying a redirect-only name, and use a dynamic
target so deep links and query strings survive. The defensive-domain Worker
performs the equivalent redirect with status 308.

## Credentials

Never record credential values or filesystem locations here.

| Credential | Scope and use | Held by |
| --- | --- | --- |
| Wrangler OAuth login | Worker scripts, routes, bindings, and read-only account inspection used for manual deployments | `kim@lean-fro.org` |
| DNS/Redirect API token | DNS edit, zone read, and Single Redirect edit on the two Palomar zones only | `kim@lean-fro.org` |
| R2 publisher Account API token | Object read/write/list on the single `palomar-public-data` bucket; no bucket administration, Worker deployment, DNS, registrar, or billing access | GitHub Actions in private `PalomarDatabase` |
| Server deployment API token | Worker version upload and promotion for `palomar-server` | GitHub Actions in `PalomarServer` |
| Registry automation token (`PALOMAR_GITHUB_TOKEN`) | Reads verification artifacts, updates private submission state, and creates and merges registration changes in the private database | GitHub Actions in private `PalomarSubmissionState` |
| Archive token (`PALOMAR_ARCHIVE_TOKEN`) | Authenticates only as `PalomarArchivist`; creates and writes native public forks in `PalomarArchive` and manages each new fork's preservation ruleset and direct creator grant. A classic token needs `public_repo` and `workflow`; GitHub requires `workflow` even for a tag whose commit contains a workflow file. | GitHub Actions in private `PalomarSubmissionState` |
| Review-engine API key (`OPENAI_API_KEY`) | Runs the private editorial review pipeline | GitHub Actions in private `PalomarSubmissionState` |

The private Database repository stores only these R2 publisher secrets:

- `CLOUDFLARE_ACCOUNT_ID`
- `R2_ACCESS_KEY_ID`
- `R2_SECRET_ACCESS_KEY`

For an R2 Account API token, select the `R2 buckets` resource scope and restrict
it to `palomar-public-data`. The required bucket permission is
`Workers R2 Storage Bucket Item Write`, which permits reading, writing, and
listing objects without bucket administration. Cloudflare's S3 mapping is:

- Access Key ID: the API token ID;
- Secret Access Key: the SHA-256 digest of the API token value.

The token-creation confirmation may show those two S3 values directly. Copy
them then: the secret is not shown again. Install the mapped values as the two
`R2_*` GitHub Actions secrets, not as Worker secrets and not in repository
variables. Rotating the parent token requires replacing both mapped secrets.

The `palomar-data` Worker itself needs no credential because its R2 access is a
binding. The publisher needs S3 credentials because it runs in GitHub Actions.

The private SubmissionState repository stores these reviewer secrets:

- `OPENAI_API_KEY`;
- `PALOMAR_GITHUB_TOKEN`;
- `PALOMAR_ARCHIVE_TOKEN`.

The exact repository permissions for `PALOMAR_GITHUB_TOKEN`, archive-account
guardrails, and rotation procedure are maintained in the
[`PalomarSubmissionState` runbook](https://github.com/PalomarRegistry/PalomarSubmissionState#secrets).
For a fine-grained archive token, select all `PalomarArchive` repositories
(including future repositories) and grant Metadata read plus Administration,
Contents, and Workflows write. GitHub documents Workflows as an additional
permission for Git-ref creation in its
[fine-grained PAT permission table](https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens#repository-permissions-for-workflows).
Do not combine the registry and archive credentials: the archive identity must
not be able to mutate `PalomarRegistry`, and the general registry automation
identity must not bypass the archive's dedicated-account check.

## Deployment and recovery

### Website

Pushes to `PalomarWeb/main` test, build, and deploy GitHub Pages. The hourly
`Published site health` workflow checks that the live site names the current
commit and that the website's own validators can load the live index and every
active public entry. It dispatches one fresh Pages deployment when the served
build is stale and fails loudly when the public projection has become
incompatible with the website contract.

### Submission server

Pushes to `PalomarServer/main` run tests, upload a Cloudflare Worker version,
and promote it. The workflow does not change its route or cron trigger.

### Review, registration, and source preservation

The private `PalomarSubmissionState` workflow runs every fifteen minutes and
may also be dispatched manually. It installs `PalomarReviewer` from `main`, runs
`palomar-review doctor`, and then advances each live submission by at most one
state transition. Review, registration, and finalization are serialized by one
workflow concurrency group.

After creating or rotating `PALOMAR_ARCHIVE_TOKEN`, sign in as the archive
account at <https://github.com/settings/tokens>. Editing an existing classic
token's scopes preserves its value; a replacement token must also replace the Actions secret at
<https://github.com/PalomarRegistry/PalomarSubmissionState/settings/secrets/actions>,
and run a preflight with new model reviews disabled:

```sh
gh workflow run reviewer.yml \
  --repo PalomarRegistry/PalomarSubmissionState \
  --ref main -f max_reviews=0
```

The `Check prerequisites` log must say
`archive token: PalomarArchivist (verified)`. This proves the installed secret's
identity and organization access without creating a synthetic test fork.
`max_reviews=0` suppresses new model reviews but does not suppress registration
or finalization of an already-ready submission; inspect the private inflight
state first if the run must be credential-only. The first real registration
additionally proves fork creation, ruleset creation, administrator demotion, tag
creation, and read-back; it fails closed before publication if any one of those
operations is unavailable.

For a newly registered record, inspect its `preservation.repositories` mapping
and verify each linked [PalomarArchive repository](https://github.com/PalomarArchive),
the repository ruleset under **Settings → Rules → Rulesets**, and the exact
record-specific tag. The six-hour source-availability workflow then checks both
the original and preserved commits. It publishes active-only status to the data
Worker, retains complete mutable state on the private `availability` branch,
and fails visibly if a preserved copy is confirmed missing twice consecutively.

### Public registry data

Pushes affecting publishable Database inputs validate the complete private
ledger, stage the filtered snapshot, upload immutable release objects, read
them back, verify their digests, and update `_current.json` last. The hourly
`Published evidence health` workflow checks every registered render and, if one
is missing, dispatches the filtered R2 publisher—not GitHub Pages.

Before changing the data Worker, deploy and smoke-test `palomar-data-staging`.
Then deploy production and verify at least:

```sh
curl --fail https://data.palomar-registry.org/healthz
curl --fail https://data.palomar-registry.org/index.json
curl --fail --head https://data.palomar-registry.org/index.json
curl -X POST -o /dev/null -w '%{http_code}\n' https://data.palomar-registry.org/index.json # expect 405
curl -o /dev/null -w '%{http_code}\n' https://data.palomar-registry.org/_current.json # expect 404
```

The obsolete Database Pages site should remain absent. Anonymous GitHub API,
raw-content, and former Pages requests for `PalomarDatabase` should all return
404, while authenticated maintainers retain Git and API access.

## Moving a hostname

Hostnames are pinned in code, CSPs, feeds, workflows, and immutable records.
For a data- or website-domain change, audit at least:

| Where | What |
| --- | --- |
| `PalomarWeb/assets/security.mjs` | `DEFAULT_DATABASE`, `DEFAULT_RENDER_BASE`, and source-availability URL |
| `PalomarWeb/assets/app.js` | canonical website and feed bases |
| `PalomarWeb/{index,entry,render,404,about}.html` | CSP `connect-src`/`frame-src`, feed links, and data links |
| `PalomarWeb/.github/workflows/*.yml` and tests | pinned public-data and website origins |
| `PalomarDatabase/worker/wrangler.jsonc` | Worker custom domain |
| `PalomarDatabase/tools/{build_feeds.py,check_published.py}` | website, feed, and public-data bases |
| `PalomarDatabase/docs/*.md` | publication and render-origin runbooks |
| `PalomarReviewer/src/palomar_reviewer/cli.py` | public website/data URLs used during registration |
| `PalomarServer/wrangler.jsonc` | website URL and submission route |

Registered entries, evidence, renders, and in-use schemas are immutable. Never
rewrite their historical URLs. Keep an old hostname serving or redirecting as
long as an immutable record names it. Feed XML regenerates, but subscribers may
retain old copies.

For a new Pages website domain: verify it at the GitHub organization, create
the complete DNS set, attach it to `PalomarWeb`, wait for the certificate, then
enforce HTTPS. For a new data domain: add it to the production Worker config,
deploy and verify the Worker, update all consumers and CSPs, deploy them, and
only then retire the old Worker custom domain. Never point either data hostname
at raw GitHub content.

## What is deliberately manual

Registrar operations, payment, organization-level GitHub Pages domain
verification, the `www` Redirect Rule, data-Worker staging/production
deployments, creation and recovery of the `PalomarArchivist` GitHub account,
archive-organization policy changes, and credential rotation are manual.
Website, server, review, registration, source preservation, filtered-data,
availability, and health-check workflows are automated. The account is
currently using the Cloudflare Workers and R2 free tiers; usage and paid-plan
triggers are tracked in the workspace `TODO.md`.
