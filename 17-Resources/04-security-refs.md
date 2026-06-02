# Security References

This guide collects CVE, compliance, hardening, and practical security reference material.

## Security References

<table>
  <thead>
    <tr>
      <th>Resource</th>
      <th>URL</th>
      <th>Purpose/Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CVE</td>
      <td>https://cve.mitre.org/</td>
      <td>The canonical naming system for publicly disclosed vulnerabilities; useful for precise tracking and reporting.</td>
    </tr>
    <tr>
      <td>NVD</td>
      <td>https://nvd.nist.gov/</td>
      <td>Adds CVSS, CPE mappings, and enrichment that helps prioritize patching and exposure reviews.</td>
    </tr>
    <tr>
      <td>CISA Known Exploited Vulnerabilities</td>
      <td>https://www.cisa.gov/known-exploited-vulnerabilities-catalog</td>
      <td>Prioritizes what attackers are actually exploiting, which is invaluable for patch triage.</td>
    </tr>
    <tr>
      <td>CIS Benchmarks</td>
      <td>https://www.cisecurity.org/cis-benchmarks/</td>
      <td>Baseline hardening guidance for operating systems, cloud services, and middleware.</td>
    </tr>
    <tr>
      <td>OWASP</td>
      <td>https://owasp.org/</td>
      <td>A broad application security reference covering common risk areas and defensive patterns.</td>
    </tr>
    <tr>
      <td>OWASP Cheat Sheet Series</td>
      <td>https://cheatsheetseries.owasp.org/</td>
      <td>High-signal implementation guidance that is faster to consult than full standards documents.</td>
    </tr>
    <tr>
      <td>SSL Labs</td>
      <td>https://www.ssllabs.com/ssltest/</td>
      <td>Tests public TLS endpoints and quickly exposes certificate, protocol, and cipher issues.</td>
    </tr>
    <tr>
      <td>Shodan</td>
      <td>https://www.shodan.io/</td>
      <td>Searches internet-exposed services; useful for inventory validation and external attack-surface discovery.</td>
    </tr>
    <tr>
      <td>VirusTotal</td>
      <td>https://www.virustotal.com/</td>
      <td>Aggregates malware scanning and reputation checks for files, URLs, and domains.</td>
    </tr>
    <tr>
      <td>Exploit Database</td>
      <td>https://www.exploit-db.com/</td>
      <td>Good for understanding exploitability, proof-of-concept maturity, and affected software patterns.</td>
    </tr>
    <tr>
      <td>MITRE ATT&amp;CK</td>
      <td>https://attack.mitre.org/</td>
      <td>Maps attacker techniques and tactics so defenders can reason about detections and mitigations.</td>
    </tr>
    <tr>
      <td>OSV.dev</td>
      <td>https://osv.dev/</td>
      <td>Modern vulnerability database with strong open-source package coverage and machine-readable data.</td>
    </tr>
    <tr>
      <td>OpenSCAP</td>
      <td>https://www.open-scap.org/</td>
      <td>Useful for compliance scanning, benchmark execution, and automated hardening verification.</td>
    </tr>
    <tr>
      <td>Lynis</td>
      <td>https://cisofy.com/lynis/</td>
      <td>A practical host auditing tool that quickly surfaces common Linux hardening gaps.</td>
    </tr>
    <tr>
      <td>Mozilla SSL Configuration Generator</td>
      <td>https://ssl-config.mozilla.org/</td>
      <td>Great for producing sane TLS settings for Nginx, Apache, and similar front ends.</td>
    </tr>
    <tr>
      <td>Have I Been Pwned</td>
      <td>https://haveibeenpwned.com/</td>
      <td>Useful for checking exposure in public breach data and improving credential hygiene practices.</td>
    </tr>
  </tbody>
</table>
