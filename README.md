# VulnWatch

**A web vulnerability scanner that crawls a target site and checks it against common security issues, then generates a polished HTML (or JSON) report.**

VulnWatch runs a set of automated checks across a target's pages — missing security headers, exposed sensitive files, basic XSS/SQLi probing, cookie flags, CORS misconfigurations, and more — and ranks everything it finds by severity.

---

## Checks Performed

- Missing or weak HTTP security headers (HSTS, CSP, X-Frame-Options, etc.)
- SSL/TLS configuration issues
- Open redirects
- Reflected XSS (basic payload-based detection)
- SQL injection probing (error-based, basic)
- Directory listing exposure
- Sensitive file/path exposure (`.env`, `.git`, backups, admin panels)
- Cookie security flags (`Secure`, `HttpOnly`, `SameSite`)
- Server/technology information disclosure
- CORS misconfiguration
- Clickjacking protection (`X-Frame-Options` / CSP `frame-ancestors`)
- Form analysis (CSRF token presence)

Findings are rated **CRITICAL / HIGH / MEDIUM / LOW / INFO**, each with a description, evidence, and a recommended fix.

---

## Requirements

- Python 3.7+
- `requests` (required)
- `beautifulsoup4` (optional, but recommended — some checks like form/CSRF analysis are limited without it)

```bash
pip install requests beautifulsoup4
```

---

## Usage

```bash
python3 web_vuln_scanner.py -u https://example.com
```

### Options

| Flag | Description | Default |
|---|---|---|
| `-u`, `--url` | Target URL (required) | none |
| `-o`, `--output` | HTML report output path | `scan_report.html` |
| `--json` | Also save a JSON report to this path | none |
| `--depth` | Crawl depth (max recommended: 3) | `1` |
| `--timeout` | Request timeout in seconds | `10` |
| `--delay` | Delay between requests in seconds | `0.3` |
| `--threads` | Concurrent threads for path scanning | `5` |
| `--no-ssl-verify` | Disable SSL certificate verification | off |
| `--proxy` | HTTP/S proxy URL (e.g. for routing through Burp) | none |
| `--user-agent` | Custom User-Agent string | built-in default |

### Examples

Basic scan:

```bash
python3 web_vuln_scanner.py -u https://example.com
```

Deeper crawl with an HTML report:

```bash
python3 web_vuln_scanner.py -u https://example.com --depth 2 -o report.html
```

Save JSON output and skip SSL verification (useful for staging targets):

```bash
python3 web_vuln_scanner.py -u https://example.com --json results.json --no-ssl-verify
```

Slow down requests and extend the timeout for a sensitive target:

```bash
python3 web_vuln_scanner.py -u https://example.com --delay 1.0 --timeout 15
```

Route traffic through a proxy (e.g. Burp Suite) to inspect requests:

```bash
python3 web_vuln_scanner.py -u https://example.com --proxy http://127.0.0.1:8080
```

---

## Output

VulnWatch always prints a color-coded summary to the console and writes an **HTML report** (default `scan_report.html`) with:

- Scan metadata (target, start/end time, pages scanned)
- A severity breakdown chart
- A full list of findings with evidence and recommended fixes
- The list of scanned URLs and any errors encountered

Pass `--json` to also get a machine-readable JSON report for feeding into other tools or CI pipelines.

---

## How It Works

1. Crawls the target starting from the given URL, up to `--depth` levels
2. Runs each check category against every discovered page
3. Probes a built-in list of sensitive paths (`.env`, `.git/HEAD`, `wp-config.php`, backup files, admin panels, etc.) — no external wordlist needed, this list is baked into the script
4. Sends lightweight XSS and SQLi payloads to detect naive reflections and SQL error messages
5. Aggregates everything into a `ScanResult`, then renders the console, HTML, and (optionally) JSON reports

---

## Limitations

- XSS and SQLi detection are **basic/heuristic** — they catch obvious, unfiltered cases, not blind or second-order vulnerabilities
- Sensitive-path checks use a fixed built-in list, not a custom wordlist (see [PathHound](#) if you need that)
- Crawl depth beyond 2–3 can be slow and noisy on large sites
- Results should be manually verified — this is a recon aid, not a replacement for manual testing

---

## Legal & Ethical Use

Only scan systems you own or have explicit written permission to test. Unauthorized scanning may be illegal and may violate a bug bounty program's rules of engagement. The author is not responsible for misuse of this tool.

---

## Author

**Oladunni Oluwaseyi**
GitHub: [@oluwaseyi-rgb](https://github.com/oluwaseyi-rgb)
LinkedIn: [Oluwaseyi Oladunni](https://www.linkedin.com/in/oluwaseyi-oladunni-6a42b1407)

---

## License

Released under the MIT License.
