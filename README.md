# CVE Lens

Search one CVE and see every public source on one page: CVSS scores (NVD, the CNA, CISA ADP,
Red Hat, SUSE, Oracle, Amazon, Ubuntu), EPSS exploit probability, CISA KEV status, CISA SSVC, affected products, fix status for
Red Hat, SUSE / SLES, Oracle Linux, Amazon Linux, AlmaLinux, Rocky Linux, Azure Linux, Ubuntu, Debian, Alpine and language packages (npm, PyPI, Maven, Go, ...), public
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

Search several CVEs at once by separating them with commas (or pasting text that contains more than
one id), or link to `?q=CVE-2024-3094,CVE-2021-44228`. The page then shows a summary table, one row
per CVE: package, CVSS score and rating, EPSS, CISA KEV, public exploit code and publish date. Package names come
from Red Hat, else from the CNA's affected products, else from NVD's CPEs. It asks only NVD, CVE.org, EPSS,
KEV, Red Hat and PoC-in-GitHub, and looks up at most 50 CVEs. NVD allows 5 requests per 30 seconds
without an API key, so the page paces its NVD requests and long lists fill in slowly. Click a column
heading to sort, or a CVE to open its full page. **Export CSV** downloads the table in its current
order, with every package name, the full summary, the CVSS vector and the raw EPSS values.

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
| SUSE | `www.suse.com/security/cve/{id}.html` (the HTML page, parsed) | SUSE severity and CVSS, overall state, status per product and package (SLES, openSUSE, images, containers), advisories |
| Oracle Linux (via vuln-list) | `raw.githubusercontent.com/aquasecurity/vuln-list` `oval/oracle/{year}/ELSA-....json` | errata, fixed package versions per Oracle Linux release |
| Amazon Linux | `explore.alas.aws.amazon.com/{id}.json` | Amazon severity and CVSS, status per release and package, ALAS advisories |
| AlmaLinux (via OSV) | `api.osv.dev/v1/vulns/ALSA-...` | errata, fixed package versions per release |
| Rocky Linux (via OSV) | `api.osv.dev/v1/vulns/RLSA-...` | errata, fixed package versions per release (partial) |
| Azure Linux (via OSV) | `api.osv.dev/v1/query` (by package name) | fixed or last affected version per release (partial, no kernel) |
| Ubuntu (via OSV) | `api.osv.dev/v1/vulns/UBUNTU-{id}` | priority, fixed versions per release, USNs |
| OSV | `api.osv.dev/v1/vulns/{id}`, `DEBIAN-{id}`, `ALPINE-{id}`, `GHSA-...` | Debian and Alpine fix status, GitHub advisories for language packages, SUSE and Debian advisory ids |
| PoC-in-GitHub | `raw.githubusercontent.com/nomi-sec/PoC-in-GitHub` | public proof-of-concept repositories, by stars |

Notes:

- `www.cisa.gov` and `ubuntu.com` do not allow browser requests from other sites (no CORS
  headers), so the page uses CISA's official GitHub copy of the KEV catalog and Canonical's data as
  published in OSV. OSV lists only Ubuntu releases with a fix or a pending fix; the Ubuntu link on
  the page shows the full status.
- SUSE's per-CVE JSON (CSAF-VEX on `ftp.suse.com`) has no CORS header, but its CVE web page does, so
  the page reads that. It is parsed as an inert document and only its text is used. If SUSE changes
  the page layout, the SUSE section goes empty until the parser is updated.
  SUSE's reply for a CVE it has no page for carries no CORS header, so the browser cannot tell that
  case from a network error; the page reports both as "no SUSE page for this CVE, or SUSE unreachable".
- `linux.oracle.com` has no CORS header and OSV has no Oracle data. Oracle errata that rebuild a Red
  Hat update keep its number (RHSA-2024:4312 is ELSA-2024-4312), as do AlmaLinux and Rocky. The page
  takes those numbers, plus ELSA links in the NVD and CVE.org references, fetches each from the
  aquasecurity/vuln-list copy of Oracle's OVAL data, and keeps an erratum only if it names the CVE.
  Oracle-only errata that nothing links to (UEK kernels, Ksplice) and unfixed releases are missed;
  the Oracle Linux link on the page shows the full status.
- AlmaLinux errata are the ALSA ids that OSV's CVE record links to. OSV's CVE record does not link
  Rocky errata, and Rocky's own API (`errata.rockylinux.org`) has no CORS header, so the page tries
  the Red Hat and AlmaLinux advisory numbers as RLSA ids in OSV and keeps the ones that name the CVE.
  OSV lacks many Rocky errata, so a missing Rocky release does not mean it is unfixed.
- Azure Linux: Microsoft's data is in OSV as `AZL-...` records, but OSV cannot be searched by CVE and
  MSRC has no CORS header. The page asks OSV for all Azure Linux records of up to four packages that
  Amazon and Red Hat name for the CVE, and keeps the records for this CVE. Kernel packages are not
  searched (the list is several MB per page), and packages with another name on Azure Linux are missed.
- The Ubuntu CVSS score is computed from the vector in OSV, which carries no number.
- The KEV catalog (about 1.8 MB) is downloaded once and kept, compacted, in the browser's
  localStorage for 12 hours. Complete answers are kept in memory for 30 minutes, for
  back/forward and the recent links; the Search button always queries the sources again.
- NVD allows 5 requests per 30 seconds per IP address without an API key. That is plenty for
  searching by hand; a key cannot be used safely from a public page.
- Red Hat's CDN caches the CORS header of the first site to fetch each CVE (no `Vary: Origin`), which
  blocks every other site. The page adds `?cl=<its own origin>` to Red Hat requests so each site gets
  its own cache entry.
- PoC-in-GitHub lists repositories that mention the CVE. They are not vetted: some are fake or
  contain malware. The page says so next to the list.
- A source that fails or is rate limited is marked at the bottom of the page; the rest still render.

## Privacy

Only the CVE id, advisory numbers derived from it, and (for the Azure Linux lookup) the names of the
affected packages are sent to the sources, from the visitor's own browser (plus the page's own origin,
which browsers send anyway and which the Red Hat request repeats as `cl`). The page sends no referrer,
loads nothing else, and its Content-Security-Policy only allows connections to the eight source hosts.
