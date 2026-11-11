# CVE Lens

Search one CVE and see every public source on one page: CVSS scores (NVD, the CNA, CISA ADP,
Red Hat), EPSS exploit probability, CISA KEV status, CISA SSVC, affected products, fix status for
Red Hat, Ubuntu, Debian, Alpine and language packages (npm, PyPI, Maven, Go, ...), public
proof-of-concept repositories, references, and links to vendor and exploit databases.

One static file, `index.html`. No backend, no build step, no dependencies: the browser queries the
sources directly. Host it anywhere that serves static files (GitHub Pages, Cloudflare Pages, S3,
nginx), or run it locally:

```bash
python3 -m http.server 8088 --bind 127.0.0.1   # then open http://127.0.0.1:8088/?q=CVE-2024-3094
```

Serve it over HTTP(S); opening it straight from disk (`file://`) is untested.

Link to a CVE with `?q=CVE-2024-3094`. Paste any text containing a CVE id into the search box to
pick the id out of it. Press `/` to focus the search box.

**Export PDF** opens the browser's print dialog; choose "Save as PDF". The printout is always light,
keeps the severity colours, and includes folded table rows (but not the full exploit repository list).

## Sources

| Source | Endpoint | Gives |
|---|---|---|
| NVD | `services.nvd.nist.gov/rest/json/cves/2.0` | CVSS v2/v3/v4, CWE, CPEs, references |
| CVE.org | `cveawg.mitre.org/api/cve/{id}` | CNA record: title, description, affected versions, CNA and CISA ADP scores, SSVC |
| FIRST EPSS | `api.first.org/data/v1/epss` | probability of exploitation in the next 30 days, percentile |
| CISA KEV | `raw.githubusercontent.com/cisagov/kev-data` | known exploitation, ransomware use, due date |
| Red Hat | `access.redhat.com/hydra/rest/securitydata/cve/{id}.json` | Red Hat severity and CVSS, fixed and affected products |
| Ubuntu (via OSV) | `api.osv.dev/v1/vulns/UBUNTU-{id}` | priority, fixed versions per release, USNs |
| OSV | `api.osv.dev/v1/vulns/{id}`, `DEBIAN-{id}`, `ALPINE-{id}`, `GHSA-...` | Debian and Alpine fix status, GitHub advisories for language packages, SUSE / AlmaLinux / Rocky advisory ids |
| PoC-in-GitHub | `raw.githubusercontent.com/nomi-sec/PoC-in-GitHub` | public proof-of-concept repositories, by stars |

Notes:

- `www.cisa.gov` and `ubuntu.com` do not allow browser requests from other sites (no CORS
  headers), so the page uses CISA's official GitHub copy of the KEV catalog and Canonical's data as
  published in OSV. OSV lists only Ubuntu releases with a fix or a pending fix; the Ubuntu link on
  the page shows the full status.
- The KEV catalog (about 1.8 MB) is downloaded once and kept, compacted, in the browser's
  localStorage for 12 hours. Complete answers are kept in memory for 30 minutes.
- NVD allows 5 requests per 30 seconds per IP address without an API key. That is plenty for
  searching by hand; a key cannot be used safely from a public page.
- Red Hat's CDN caches the CORS header of the first site to fetch each CVE (no `Vary: Origin`), which
  blocks every other site. The page adds `?cl=<its own origin>` to Red Hat requests so each site gets
  its own cache entry.
- PoC-in-GitHub lists repositories that mention the CVE. They are not vetted: some are fake or
  contain malware. The page says so next to the list.
- A source that fails or is rate limited is marked at the bottom of the page; the rest still render.

## Privacy

Only the CVE id is sent to the sources, from the visitor's own browser (plus the page's own origin,
which browsers send anyway and which the Red Hat request repeats as `cl`). The page sends no referrer,
loads nothing else, and its Content-Security-Policy only allows connections to the six source hosts.
