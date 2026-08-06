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
| `www.palomar-registry.org` | Redirects to the apex | Cloudflare, at the edge | Live, 301, path and query preserved |
| `data.palomar-registry.org` | Render bundles and RSS feeds | GitHub Pages, `PalomarDatabase` | Live, HTTPS enforced |
| `submit.palomar-registry.org` | The submission server | Cloudflare Worker, `PalomarServer` | Live |
| `palomarregistry.org`, `www` | Redirects to `palomar-registry.org` | Cloudflare, at the edge | Live, 308, separate zone |
| `palomarregistry.github.io/PalomarWeb/` | Nothing directly | GitHub Pages | 301 to `palomar-registry.org` |
| `palomarregistry.github.io/PalomarDatabase/` | Nothing directly | GitHub Pages | 301 to `data.palomar-registry.org` |

Cloudflare is registrar and DNS for the names GitHub serves, hosts `submit` as a
Worker, and terminates TLS for the redirect-only names, `www` and the whole
`palomarregistry.org` zone. Only the apex and `data` reach GitHub at all.

`palomarregistry.org`, without the hyphen, is a defensive registration held so
the obvious misspelling of the real domain does not go somewhere else. It is a
separate Cloudflare zone and serves nothing; every name in it redirects.

The render origin is `data.palomar-registry.org`. Older notes call it
`renders.`, which does not resolve.

Raw record data is fetched from
`raw.githubusercontent.com/PalomarRegistry/PalomarDatabase/main/index.json`,
not from Pages. That is deliberate: the database is the source of truth and is
read at its canonical location, while Pages serves only derived artifacts.

## DNS records

| Type | Name | Target | Proxy |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153`, `.109.153`, `.110.153`, `.111.153` | **DNS only (grey cloud)** |
| AAAA | `@` | `2606:50c0:8000::153` through `8003::153` | **DNS only (grey cloud)** |
| CNAME | `data` | `palomarregistry.github.io` | **DNS only (grey cloud)** |
| CNAME | `www` | `palomarregistry.github.io` | Proxied (orange cloud) |
| AAAA | `submit` | `100::` | Proxied (orange cloud) |
| TXT | `_github-pages-challenge-palomarregistry` | the org verification token | DNS only |

The proxy setting is not cosmetic, but the rule is narrower than "always grey".
**A name must be grey-clouded when GitHub has to receive the traffic and
terminate TLS for it**, which is true of the apex and `data`. Proxy one of
those and GitHub cannot complete its certificate challenge, and you get
certificate errors or a redirect loop rather than a clean failure.

The two proxied names are proxied precisely because GitHub is not involved.
`submit` is a Cloudflare Worker; wrangler creates that record itself, and the
`100::` placeholder target is normal and should not be edited by hand. `www`
is answered by a Cloudflare Redirect Rule at the edge, so its CNAME target is
vestigial: the origin is never contacted. See *Certificates and HTTPS*.

The apex is eight address records rather than one alias because ordinary DNS
cannot put a CNAME at a zone apex. Cloudflare could hide that with CNAME
flattening, but this zone uses GitHub's published A and AAAA set explicitly,
which is what GitHub documents. If GitHub ever changes that address set, these
are the records that have to change.

The `palomarregistry.org` zone holds two records, `@` and `www`, both
`AAAA 100::` and both proxied, which is Cloudflare's documented shape for a
zone that only redirects.

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

Two certificate authorities are in play, and which one serves a name depends on
whether that name is proxied. GitHub Pages issues from Let's Encrypt for the
apex and `data`. Cloudflare's Universal SSL, currently from Google Trust
Services, covers `palomar-registry.org` and `*.palomar-registry.org` and is what
`www` and `submit` present. Neither needs installing or renewing by hand.

**Enforce HTTPS is a separate switch from having a certificate.** A host can
have a valid, approved certificate and still serve plaintext on port 80 without
redirecting. It is `https_enforced` on the Pages API, it can only be turned on
once the certificate is approved, and it has to be set explicitly on each
repository:

```
gh api -X PUT repos/PalomarRegistry/<repo>/pages -F https_enforced=true
```

When checking whether it took effect, do not test `/`. Pages sits behind Fastly
with `max-age=600`, so the root can serve a cached pre-enforcement `200` for ten
minutes after the switch flips. Request a path that cannot be in the cache:

```
curl -sSI "http://palomar-registry.org/nonexistent-$RANDOM.html"   # expect 301
```

**`www` is redirected by Cloudflare, not served by GitHub.** It is a proxied
record answered by a Redirect Rule in the `http_request_dynamic_redirect` phase:

```
expression:  http.host eq "www.palomar-registry.org"
target:      concat("https://palomar-registry.org", http.request.uri.path)
status:      301, preserve_query_string: true
```

The rule fires at the edge, so the origin is never contacted and GitHub is out
of the `www` path. Cloudflare's wildcard already covers the name, so no
certificate has to be issued for it. `palomarregistry.org` uses the same pattern
in its own zone.

Two rules for building one of these. Create the rule *before* flipping the
record to proxied, so there is no window where a proxied name has no redirect
behind it. And use a dynamic target rather than a static one, or every deep link
collapses to the front page; record URLs are permanent, so that is not
recoverable.

Do not put a redirect-only name on Pages instead. GitHub issues a certificate
for the names configured when it issues, and there is no reliable, zero-downtime
way to make it reissue for a name added later: detach and reattach, which is the
documented retry, does not dependably do it, and pointing the custom domain at
the name and back takes the main site down for the duration. A name whose only
job is to redirect does not need an origin at all.

GitHub's health check reports `www` with `is_pointed_to_github_pages_ip: false`,
`is_non_github_pages_ip_present: true` and `is_https_eligible: false`, alongside
`responds_to_https: true` and no error. That combination is expected, not a
fault. If CAA records are ever added to the zone, they must permit Cloudflare's
CA as well as Let's Encrypt, or `www` and `submit` will fail to renew.

To check what a name actually presents, ask the connection rather than any API:

```
curl -sS -o /dev/null -w '%{http_code} %{ssl_verify_result}\n' https://www.palomar-registry.org/
python3 -c "import ssl,socket;c=ssl.create_default_context();c.check_hostname=False;s=c.wrap_socket(socket.create_connection(('www.palomar-registry.org',443)),server_hostname='www.palomar-registry.org');print([v for _,v in s.getpeercert()['subjectAltName']])"
```

Expect a lag after any change of this kind. The old grey-cloud address stays in
resolver caches for its TTL, and during that window a fraction of requests still
reach GitHub and fail verification. It clears on its own; measure with repeated
requests rather than one.

The lesson for any future Pages host: **create the DNS record first, then attach
the custom domain.** Doing it in the other order produces a certificate that is
correct for what was configured and wrong for what people will type, and that is
much harder to undo than to avoid.

## Credentials

| Credential | Scope | Held where |
| --- | --- | --- |
| wrangler OAuth login | workers, workers_kv, workers_routes, workers_scripts, workers_tail, account read, user read | `~/.config/.wrangler/config/default.toml` |
| Cloudflare API token | Zone · DNS · Edit, Zone · Zone · Read, and Zone · Single Redirect · Edit, on **both** `palomar-registry.org` and `palomarregistry.org` | `~/.palomar-cf-token`, mode 600 |

The wrangler login deliberately cannot touch DNS. The separate token
deliberately cannot touch Workers. Neither can spend money or transfer the
domain.

The token can repoint any hostname in either zone and rewrite the redirect rules
both zones depend on. It cannot read or change zone SSL/TLS settings, touch
Worker routes or scripts, reach account-level rulesets such as Bulk Redirects,
or reach the registrar or billing. So it cannot disable Universal SSL, cannot
take down `submit`, and cannot transfer or renew a domain. The blast radius is
DNS and redirects, in two zones.

Single Redirect is the dashboard name for what the API calls the
`http_request_dynamic_redirect` phase.

To test what a token may do without changing anything, send a deliberately
invalid payload. A missing permission gives `Authentication error` or
`Unauthorized to access requested resource`; a permitted request fails
validation instead, naming the offending field.

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

1. Verify the domain for the GitHub organisation **first**, at
   Settings → Pages → verified domains. Do this before any DNS points at
   Pages. Without verification, someone else can attach an unclaimed subdomain
   of a domain you own to *their* Pages site, and publishing the DNS first is
   what makes such a takeover work rather than merely possible.
2. Then create **every** DNS record the domain needs, grey-clouded, and confirm
   each resolves, including any name you want covered by the certificate.
3. Attach the custom domain in the repository's Pages settings and wait for
   the certificate. Enforce HTTPS becomes available only once it is issued,
   and is a separate switch that has to be turned on deliberately.
4. Ship the code changes from the table above.

Steps 1 to 3 are ordered the way they are for two different reasons that pull
in the same direction: verification first closes the takeover window, and
complete DNS before attaching is what gets every name into the certificate.
A name that only ever redirects does not need to be in this sequence at all,
and is simpler as a Cloudflare Redirect Rule.

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
