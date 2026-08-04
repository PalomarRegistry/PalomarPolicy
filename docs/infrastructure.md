# Palomar infrastructure

Last updated 2026-08-04.

This is the durable record of where Palomar runs, so that moving it is a
checklist rather than an excavation. If you are changing the domain, read the
last section first: the hard part is not DNS, it is the list of places a
hostname is written down.

## Accounts and ownership

| Thing | Where | Notes |
| --- | --- | --- |
| Repositories | GitHub org [`PalomarRegistry`](https://github.com/PalomarRegistry) | Moved from the `kim-em` personal account on 2026-08-04. Base member permission is `read`. |
| Domain | `palomar-registry.org`, registered at Cloudflare Registrar | Registrar and DNS are the same account. |
| DNS and Workers | Cloudflare account `d789bf36d237e0cb313be59b927c82bd` | Zone `f05ebb1809990a5d27e6d6a7d0d1ae85`, nameservers `joyce`/`matias.ns.cloudflare.com`. |
| Static hosting | GitHub Pages | Not Cloudflare, despite the domain being there. |

The old `kim-em/Palomar*` repository names **must stay reserved forever**.
Recreating a repository at an old name permanently destroys that name's
redirects, and three published records cite those names in immutable fields.

## Hostnames

| Host | Serves | Hosted by | Status |
| --- | --- | --- | --- |
| `palomarregistry.github.io/PalomarWeb/` | The website | GitHub Pages, `PalomarWeb` | Live |
| `palomarregistry.github.io/PalomarDatabase/` | Render bundles and RSS feeds | GitHub Pages, `PalomarDatabase` | Live |
| `renders.palomar-registry.org` | Render bundles, on their own origin | GitHub Pages, `PalomarDatabase` | DNS live, Pages custom domain not yet attached |
| `palomar-registry.org`, `www` | The website | Planned: Cloudflare Workers Assets | Not built |
| `submit.palomar-registry.org` | The submission server | Planned: Worker custom domain | Not built |

Raw record data is fetched from
`raw.githubusercontent.com/PalomarRegistry/PalomarDatabase/main/index.json`,
not from Pages. That is deliberate: the database is the source of truth and is
read at its canonical location, while Pages serves only derived artifacts.

## DNS records

| Type | Name | Target | Proxy |
| --- | --- | --- | --- |
| CNAME | `renders` | `palomarregistry.github.io` | **DNS only (grey cloud)** |

The proxy setting is not cosmetic. Proxied, GitHub cannot complete its
certificate challenge for the custom domain, and the result is certificate
errors or a redirect loop rather than a clean failure. Any hostname delegated
to GitHub Pages must be grey-clouded.

## Why renders get their own origin

`PalomarDatabase/docs/render-origin.md` has the full argument. In short: a
render bundle is untrusted output derived from a submitter's Lean source. It
is embedded in an iframe with `sandbox="allow-scripts"` and deliberately
without `allow-same-origin`, so it receives an opaque origin. While the site
and the bundles share `palomarregistry.github.io`, that sandbox attribute is
the *only* thing separating them. A dedicated hostname makes it one layer
among several rather than the whole defence.

Consequence to remember: a GitHub Pages custom domain **drops the project path
prefix**. Renders move from
`palomarregistry.github.io/PalomarDatabase/renders/...` to
`renders.palomar-registry.org/renders/...`. The base and the CSP must change in
the same window.

## Credentials

| Credential | Scope | Held where |
| --- | --- | --- |
| wrangler OAuth login | workers, workers_kv, workers_routes, workers_scripts, workers_tail, account read, user read | `~/.config/.wrangler/config/default.toml` |
| Cloudflare API token | Zone · DNS · Edit, and Zone · Zone · Read, on `palomar-registry.org` only | `~/.palomar-cf-token`, mode 600 |

The wrangler login deliberately cannot touch DNS. The separate token
deliberately cannot touch Workers. Neither can spend money or transfer the
domain.

## Moving to a different domain

DNS is the easy part. What actually costs time is that hostnames are written
into code, into content security policies, and into artifacts. Work through
these in order.

### 1. Things that must change together

| Where | What |
| --- | --- |
| `PalomarWeb/assets/security.mjs` | `DEFAULT_DATABASE`, `DEFAULT_RENDER_BASE` |
| `PalomarWeb/assets/app.js` | `CANONICAL_WEB_BASE`, `FEED_BASE`, `DATABASE_SOURCE_BASE` |
| `PalomarWeb/{index,entry,render,404,about}.html` | CSP `frame-src`, and the RSS `<link rel="alternate">` hrefs |
| `PalomarWeb/tests/{security.test.mjs,rendering.test.js,site.spec.js,fixture_server.py}` | fixtures assert the exact hosts |
| `PalomarDatabase/tools/build_feeds.py` | `WEB_BASE`, `FEED_BASE` |
| `PalomarReviewer/src/palomar_reviewer/cli.py` | `WEB_URL` |

The CSP is the one that fails least helpfully. Get `frame-src` wrong and
renders silently do not display, with the reason only visible in the browser
console.

### 2. Things that must not change

Published records under `PalomarDatabase/entries/`, their `evidence/` and
`legacy-evidence/` bundles, and any frozen `schema-vN.json` are immutable byte
for byte. They contain absolute URLs and they keep containing them, because
they are historical statements that were true when made. Rewriting them is
indistinguishable from fabricating them.

This means **old hostnames must keep resolving, or the record links rot.**
That is an argument for owning a domain rather than depending on a hosting
provider's namespace, and it is the reason a domain move is not free even
after the code is updated.

### 3. Feeds already published

`build_feeds.py` bakes absolute URLs into `feed.xml` and the per-subject
feeds. Feeds regenerate on every database change, so they self-heal, but
anything a subscriber already fetched keeps the old links.

### 4. Ordering

1. Create the DNS record, grey-clouded, and confirm it resolves.
2. Verify the domain for the GitHub organisation, at
   Settings → Pages → verified domains, before attaching it. Without
   verification, someone else can attach an unclaimed subdomain of a domain
   you own to *their* Pages site.
3. Attach the custom domain in the repository's Pages settings and wait for
   the certificate. Enforce HTTPS becomes available only once it is issued.
4. Ship the code changes from the table above.

Steps 3 and 4 cannot be simultaneous, so plan for a window where renders do
not display. It is short and pre-launch it does not matter; after launch,
land the code change behind a fallback that tries both origins instead.

## What is deliberately not automated

Registrar operations, payment, and org-level GitHub Pages domain verification
have no API path we use, and all three are rare enough that a documented
manual step is better than a credential with the authority to perform them.
