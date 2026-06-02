# Essential RFCs

This guide collects the essential RFC table with descriptions and practical context.

## Essential RFCs Table

<table>
  <thead>
    <tr>
      <th>RFC Number</th>
      <th>Title</th>
      <th>One-line description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>768</td>
      <td>User Datagram Protocol</td>
      <td>The original UDP definition; useful for understanding lightweight, connectionless transport.</td>
    </tr>
    <tr>
      <td>791</td>
      <td>Internet Protocol</td>
      <td>Defines IPv4 packet format and forwarding fundamentals that underpin almost every network discussion.</td>
    </tr>
    <tr>
      <td>792</td>
      <td>Internet Control Message Protocol</td>
      <td>Explains ICMP messages behind ping, unreachable notices, and many path diagnostics.</td>
    </tr>
    <tr>
      <td>793</td>
      <td>Transmission Control Protocol</td>
      <td>The classic TCP specification covering sequencing, retransmission, flags, and connection state.</td>
    </tr>
    <tr>
      <td>826</td>
      <td>Address Resolution Protocol</td>
      <td>Shows how IPv4 neighbors map IP addresses to MAC addresses on local networks.</td>
    </tr>
    <tr>
      <td>1034</td>
      <td>Domain Names - Concepts and Facilities</td>
      <td>Core DNS concepts reference that explains hierarchy, zones, delegation, and naming design.</td>
    </tr>
    <tr>
      <td>1035</td>
      <td>Domain Names - Implementation and Specification</td>
      <td>The wire-format and record-type details needed for serious DNS troubleshooting.</td>
    </tr>
    <tr>
      <td>1122</td>
      <td>Requirements for Internet Hosts - Communication Layers</td>
      <td>Host behavior requirements that clarify many implementation expectations beyond base protocol RFCs.</td>
    </tr>
    <tr>
      <td>1191</td>
      <td>Path MTU Discovery</td>
      <td>Important for diagnosing fragmentation issues and mysterious packet loss on complex paths.</td>
    </tr>
    <tr>
      <td>1918</td>
      <td>Address Allocation for Private Internets</td>
      <td>Defines private IPv4 address ranges used across internal networks and NAT deployments.</td>
    </tr>
    <tr>
      <td>2131</td>
      <td>Dynamic Host Configuration Protocol</td>
      <td>The key DHCP document for leases, discovery, offer, request, and acknowledgment flow.</td>
    </tr>
    <tr>
      <td>2132</td>
      <td>DHCP Options and BOOTP Vendor Extensions</td>
      <td>Explains DHCP option fields that drive gateways, DNS, PXE, and many client behaviors.</td>
    </tr>
    <tr>
      <td>2328</td>
      <td>OSPF Version 2</td>
      <td>Useful when learning link-state routing, areas, LSAs, and internal route convergence.</td>
    </tr>
    <tr>
      <td>2616</td>
      <td>Hypertext Transfer Protocol - HTTP/1.1</td>
      <td>Historic HTTP/1.1 reference still encountered in legacy docs, proxies, and code comments.</td>
    </tr>
    <tr>
      <td>2865</td>
      <td>Remote Authentication Dial In User Service</td>
      <td>The main RADIUS authentication reference for network access and AAA systems.</td>
    </tr>
    <tr>
      <td>3207</td>
      <td>SMTP Service Extension for Secure SMTP over TLS</td>
      <td>Describes STARTTLS for SMTP, which matters when troubleshooting secure mail delivery.</td>
    </tr>
    <tr>
      <td>3411</td>
      <td>Architecture for SNMP Management Frameworks</td>
      <td>Good context for understanding how SNMP systems are structured and secured.</td>
    </tr>
    <tr>
      <td>3704</td>
      <td>Ingress Filtering for Multihomed Networks</td>
      <td>Best current practice for source address validation and anti-spoofing controls.</td>
    </tr>
    <tr>
      <td>4251</td>
      <td>The Secure Shell Protocol Architecture</td>
      <td>Top-level SSH architecture reference that frames the rest of the SSH protocol suite.</td>
    </tr>
    <tr>
      <td>4252</td>
      <td>The Secure Shell Authentication Protocol</td>
      <td>Explains how SSH authentication methods work, including public key and password flows.</td>
    </tr>
    <tr>
      <td>4253</td>
      <td>The Secure Shell Transport Layer Protocol</td>
      <td>Covers key exchange, encryption, MACs, and transport-level SSH security behavior.</td>
    </tr>
    <tr>
      <td>4254</td>
      <td>The Secure Shell Connection Protocol</td>
      <td>Defines channels, shell sessions, exec requests, forwarding, and multiplexing in SSH.</td>
    </tr>
    <tr>
      <td>4271</td>
      <td>A Border Gateway Protocol 4 (BGP-4)</td>
      <td>The core interdomain routing reference for AS paths, policies, and route exchange.</td>
    </tr>
    <tr>
      <td>4291</td>
      <td>IPv6 Addressing Architecture</td>
      <td>Explains IPv6 address structure, scopes, and interface identifier expectations.</td>
    </tr>
    <tr>
      <td>4443</td>
      <td>ICMPv6 for the Internet Protocol Version 6</td>
      <td>Critical for understanding IPv6 diagnostics and control-plane message behavior.</td>
    </tr>
    <tr>
      <td>4861</td>
      <td>Neighbor Discovery for IP version 6</td>
      <td>Explains IPv6 router discovery, address resolution, and reachability detection.</td>
    </tr>
    <tr>
      <td>5246</td>
      <td>The Transport Layer Security (TLS) Protocol Version 1.2</td>
      <td>The key TLS 1.2 document used in compatibility, compliance, and legacy support cases.</td>
    </tr>
    <tr>
      <td>5321</td>
      <td>Simple Mail Transfer Protocol</td>
      <td>The main SMTP standard for modern mail transfer, relaying, and server behavior.</td>
    </tr>
    <tr>
      <td>5322</td>
      <td>Internet Message Format</td>
      <td>Defines email message headers and structure, useful for mail debugging and parsing.</td>
    </tr>
    <tr>
      <td>6762</td>
      <td>Multicast DNS</td>
      <td>Important for understanding local service discovery and why mDNS traffic appears on LANs.</td>
    </tr>
    <tr>
      <td>7540</td>
      <td>Hypertext Transfer Protocol Version 2 (HTTP/2)</td>
      <td>Explains streams, frames, and multiplexing that affect latency and application debugging.</td>
    </tr>
    <tr>
      <td>7858</td>
      <td>Specification for DNS over Transport Layer Security</td>
      <td>Defines DNS over TLS, useful when securing recursive resolver traffic.</td>
    </tr>
    <tr>
      <td>8200</td>
      <td>Internet Protocol, Version 6 (IPv6) Specification</td>
      <td>The current IPv6 base specification and a must-have reference for dual-stack networks.</td>
    </tr>
    <tr>
      <td>8446</td>
      <td>The Transport Layer Security (TLS) Protocol Version 1.3</td>
      <td>Explains the modern TLS handshake, cipher suites, and improved security defaults.</td>
    </tr>
    <tr>
      <td>8484</td>
      <td>DNS Queries over HTTPS (DoH)</td>
      <td>Useful when analyzing encrypted DNS traffic and resolver privacy designs.</td>
    </tr>
    <tr>
      <td>9000</td>
      <td>QUIC: A UDP-Based Multiplexed and Secure Transport</td>
      <td>Defines the transport behind much of HTTP/3 and modern low-latency web traffic.</td>
    </tr>
    <tr>
      <td>9110</td>
      <td>HTTP Semantics</td>
      <td>The modern HTTP semantics reference that supersedes older split or legacy documents.</td>
    </tr>
    <tr>
      <td>9114</td>
      <td>HTTP/3</td>
      <td>Defines HTTP over QUIC and matters for CDN, browser, and edge troubleshooting.</td>
    </tr>
    <tr>
      <td>9293</td>
      <td>Transmission Control Protocol (TCP)</td>
      <td>The modern TCP consolidation RFC that refreshes and updates earlier TCP specifications.</td>
    </tr>
  </tbody>
</table>
