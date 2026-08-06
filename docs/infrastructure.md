# Palomar infrastructure

Last updated 2026-08-06.

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
redirects. No record cites those names today, because the registry is empty,
but a record's URLs are immutable from the first one onwards, so the cost of
losing a redirect only ever grows.

## Hostnames

| Host | Serves | Hosted by | Status |
| --- | --- | --- | --- |
| `palomar-registry.org` | The website | GitHub Pages, `PalomarWeb` | Live, HTTPS enforced |
| `www.palomar-registry.org` | Redirects to the apex | GitHub Pages, `PalomarWeb` | DNS live; read *Certificates* below before trusting it |
| `data.palomar-registry.org` | Render bundles and RSS feeds | GitHub Pages, `PalomarDatabase` | Live, HTTPS enforced |
| `submit.palomar-registry.org` | The submission server | Cloudflare Worker, `PalomarServer` | Live |
| `palomarregistry.github.io/PalomarWeb/` | Nothing directly | GitHub Pages | 301 to `palomar-registry.org` |
| `palomarregistry.github.io/PalomarDatabase/` | Nothing directly | GitHub Pages | 301 to `data.palomar-registry.org` |

The website is served by GitHub Pages, not by Cloudflare Workers Assets. An
earlier version of this document listed Workers as the plan for the apex; that
was never built, and attaching the Pages custom domain made it unnecessary.
Cloudflare's role here is registrar and DNS only.

The render origin is `data.palomar-registry.org`. It was called `renders.` while
it was still being planned, and that name appears in older notes; it was never
attached and no longer resolves.

Raw record data is fetched from
`raw.githubusercontent.com/PalomarRegistry/PalomarDatabase/main/index.json`,
not from Pages. That is deliberate: the database is the source of truth and is
read at its canonical location, while Pages serves only derived artifacts.

## DNS records

| Type | Name | Target | Proxy |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153`, `.109.153`, `.110.153`, `.111.153` | **DNS only (grey cloud)** |
| AAAA | `@` | `2606:50c0:8000::153` through `8003::153` | **DNS only (grey cloud)** |
| CNAME | `www` | `palomarregistry.github.io` | **DNS only (grey cloud)** |
| CNAME | `data` | `palomarregistry.github.io` | **DNS only (grey cloud)** |
| AAAA | `submit` | `100::` | Proxied (orange cloud) |
| TXT | `_github-pages-challenge-palomarregistry` | the org verification token | DNS only |

The proxy setting is not cosmetic. Proxied, GitHub cannot complete its
certificate challenge for the custom domain, and the result is certificate
errors or a redirect loop rather than a clean failure. Any hostname delegated
to GitHub Pages must be grey-clouded.

`submit` is the exception, and is proxied because it is a Cloudflare Worker
custom domain rather than a Pages site. Wrangler creates that record itself;
the `100::` placeholder target is normal and should not be edited by hand.

The apex cannot be a CNAME, which is why it is eight address records rather
than one alias. If GitHub ever changes the Pages address set, these are the
records that have to change.

## Why renders get their own origin

`PalomarDatabase/docs/render-origin.md` has the full argument. In short: a
render bundle is untrusted output derived from a submitter's Lean source. It
is embedded in an iframe with `sandbox="allow-scripts"` and deliberately
without `allow-same-origin`, so it receives an opaque origin. While the site
and the bundles share `palomarregistry.github.io`, that sandbox attribute is
the *only* thing separating them. A dedicated hostname makes it one layer
among several rather than the whole defence.

Consequence to remember: a GitHub Pages custom domain **drops the project path
prefix**. Renders moved from
`palomarregistry.github.io/PalomarDatabase/renders/...` to
`data.palomar-registry.org/renders/...`. The base and the CSP had to change in
the same window, and would again for any future move.

## Certificates and HTTPS

GitHub Pages issues the certificates, from Let's Encrypt, per repository. There
is no certificate to install or renew by hand, and Cloudflare is not terminating
TLS for any Pages host, so its certificate settings are irrelevant here.

Two things about this are worth knowing before they cost an afternoon.

**Enforce HTTPS is a separate switch from having a certificate.** A host can
have a valid, approved certificate and still serve plaintext on port 80 without
redirecting. It is `https_enforced` on the Pages API, off by default, and it can
only be turned on once the certificate is approved. It is now on for both
`PalomarWeb` and `PalomarDatabase`:

```
gh api -X PUT repos/PalomarRegistry/<repo>/pages -F https_enforced=true
```

When checking whether it took effect, do not test `/`. Pages sits behind Fastly
with `max-age=600`, so the root can serve a cached pre-enforcement `200` for ten
minutes after the switch flips. Request a path that cannot be in the cache:

```
curl -sSI "http://palomar-registry.org/nonexistent-$RANDOM.html"   # expect 301
```

**The apex certificate does not cover `www`.** GitHub issues a certificate for
the custom domain exactly as configured, and attempts a companion one for the
`www` variant only when the `www` record already resolves to the Pages site at
the moment the domain is attached. Here the `www` CNAME was created after the
apex certificate had been issued, so `www` was left being served the default
`*.github.io` wildcard, which does not match it, and browsers show a full-page
interstitial rather than a redirect.

Detaching and reattaching the custom domain does **not** fix this on its own: if
the existing certificate is still valid, GitHub reuses it, the API state never
leaves `approved`, and nothing is re-requested. The observable symptom of a real
re-issue is the state passing through `provisioning`.

To check what is actually covered, ask the connection rather than the API, since
the API reports only the primary certificate:

```
curl -sS -o /dev/null -w '%{http_code} %{ssl_verify_result}\n' https://www.palomar-registry.org/
python3 -c "import ssl,socket;c=ssl.create_default_context();c.check_hostname=False;s=c.wrap_socket(socket.create_connection(('www.palomar-registry.org',443)),server_hostname='www.palomar-registry.org');print([v for _,v in s.getpeercert()['subjectAltName']])"
```

The lesson for any future host: **create the DNS record first, then attach the
custom domain.** Doing it in the other order produces a certificate that is
correct for what was configured and wrong for what people will type.

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

Registered records under `PalomarDatabase/entries/`, their `evidence/`
bundles, and `schema-v1.json` are immutable byte for byte once written. They
contain absolute URLs and they keep containing them, because they are
historical statements that were true when made. Rewriting them is
indistinguishable from fabricating them.

The registry is empty today, so nothing has rotted yet and a move now is
cheap. That stops being true with the first record. **From then on, old
hostnames must keep resolving or the record links rot.** That is an argument
for owning a domain rather than depending on a hosting provider's namespace,
and it is the reason a domain move is not free even after the code is
updated.

### 3. Feeds already published

`build_feeds.py` bakes absolute URLs into `feed.xml` and the per-subject
feeds. Feeds regenerate on every database change, so they self-heal, but
anything a subscriber already fetched keeps the old links.

### 4. Ordering

1. Create **every** DNS record for the new domain, grey-clouded, and confirm
   each resolves. This includes `www` if you intend `www` to work. Doing this
   before the next step is what makes the certificate cover the right names;
   see *Certificates and HTTPS* for what it costs to get this order wrong.
2. Verify the domain for the GitHub organisation, at
   Settings → Pages → verified domains, before attaching it. Without
   verification, someone else can attach an unclaimed subdomain of a domain
   you own to *their* Pages site.
3. Attach the custom domain in the repository's Pages settings and wait for
   the certificate. Enforce HTTPS becomes available only once it is issued,
   and is a separate switch that has to be turned on deliberately.
4. Ship the code changes from the table above.

Steps 3 and 4 cannot be simultaneous, so plan for a window where renders do
not display. It is short and pre-launch it does not matter; after launch,
land the code change behind a fallback that tries both origins instead.

For `palomar-registry.org` itself, steps 1 and 2 are done: the organisation is
verified, and both `PalomarWeb` and `PalomarDatabase` report
`protected_domain_state: verified`.

## What is deliberately not automated

Registrar operations, payment, and org-level GitHub Pages domain verification
have no API path we use, and all three are rare enough that a documented
manual step is better than a credential with the authority to perform them.
