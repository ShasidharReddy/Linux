# Networking References

This guide collects networking RFCs, calculators, diagnostics, and address-planning references.

## Networking References

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
      <td>RFC 1918</td>
      <td>https://datatracker.ietf.org/doc/html/rfc1918</td>
      <td>Defines private IPv4 ranges; essential for planning LANs, VPNs, and NAT boundaries correctly.</td>
    </tr>
    <tr>
      <td>cidr.xyz</td>
      <td>https://cidr.xyz</td>
      <td>A visual CIDR calculator that makes subnet sizing and address range math fast and intuitive.</td>
    </tr>
    <tr>
      <td>subnet-calculator.com</td>
      <td>https://www.subnet-calculator.com/</td>
      <td>Helpful when you need quick subnet masks, host counts, wildcard masks, and broadcast calculations.</td>
    </tr>
    <tr>
      <td>IANA Service Names and Port Numbers</td>
      <td>https://www.iana.org/assignments/service-names-port-numbers</td>
      <td>The authoritative reference for well-known and registered ports; useful for firewall and audit work.</td>
    </tr>
    <tr>
      <td>RFC 791</td>
      <td>https://datatracker.ietf.org/doc/html/rfc791</td>
      <td>The classic Internet Protocol specification; valuable for understanding packet structure and routing basics.</td>
    </tr>
    <tr>
      <td>RFC 793</td>
      <td>https://datatracker.ietf.org/doc/html/rfc793</td>
      <td>The original TCP specification; still useful when learning connection setup, teardown, and flags.</td>
    </tr>
    <tr>
      <td>RFC 768</td>
      <td>https://datatracker.ietf.org/doc/html/rfc768</td>
      <td>The foundational UDP reference; good for understanding connectionless transport and checksum behavior.</td>
    </tr>
    <tr>
      <td>RFC 2616</td>
      <td>https://datatracker.ietf.org/doc/html/rfc2616</td>
      <td>Historic HTTP/1.1 reference; useful when reading legacy documentation and older proxy behavior notes.</td>
    </tr>
    <tr>
      <td>RFC 7540</td>
      <td>https://datatracker.ietf.org/doc/html/rfc7540</td>
      <td>Defines HTTP/2; helpful for understanding multiplexing, streams, and modern web performance behavior.</td>
    </tr>
    <tr>
      <td>RFC 5246</td>
      <td>https://datatracker.ietf.org/doc/html/rfc5246</td>
      <td>TLS 1.2 reference; still relevant when auditing older systems and compatibility requirements.</td>
    </tr>
    <tr>
      <td>RFC 8446</td>
      <td>https://datatracker.ietf.org/doc/html/rfc8446</td>
      <td>TLS 1.3 reference; important for modern encryption, cipher negotiation, and handshake troubleshooting.</td>
    </tr>
    <tr>
      <td>MXToolbox</td>
      <td>https://mxtoolbox.com/</td>
      <td>Excellent for DNS, MX, SMTP, blacklist, and mail-delivery checks during incident response.</td>
    </tr>
    <tr>
      <td>WhatIsMyIP</td>
      <td>https://www.whatismyip.com/</td>
      <td>Quickly confirms your public IP address; handy when checking NAT, VPN, or ISP path changes.</td>
    </tr>
    <tr>
      <td>IPinfo</td>
      <td>https://ipinfo.io/</td>
      <td>Provides ASN, geolocation, company, and network ownership data that speeds up triage and enrichment.</td>
    </tr>
    <tr>
      <td>Hurricane Electric BGP Toolkit</td>
      <td>https://bgp.he.net/</td>
      <td>Useful for ASN lookups, prefix visibility, and understanding external routing announcements.</td>
    </tr>
    <tr>
      <td>DNSViz</td>
      <td>https://dnsviz.net/</td>
      <td>Visualizes DNS and DNSSEC behavior, making delegation and signing problems much easier to spot.</td>
    </tr>
  </tbody>
</table>
