# Linux Resources & Bookmarks Guide

A curated, operator-friendly collection of Linux, networking, SRE, security, container, and automation references.

This guide is intentionally organized as tables so you can scan quickly, bookmark the essentials, and return to the exact reference you need during troubleshooting, design, or learning.

---

## 1. Networking References

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

## 2. SRE & Uptime

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
      <td>uptime.is</td>
      <td>https://uptime.is/</td>
      <td>A simple SLA calculator that translates availability targets into allowed downtime, making SLOs concrete.</td>
    </tr>
    <tr>
      <td>Google SRE Book</td>
      <td>https://sre.google/sre-book/table-of-contents/</td>
      <td>The foundational SRE text for SLIs, SLOs, toil reduction, alerting, and operational maturity.</td>
    </tr>
    <tr>
      <td>Google SRE Workbook</td>
      <td>https://sre.google/workbook/table-of-contents/</td>
      <td>Practical companion to the SRE Book with applied guidance for implementation and day-two operations.</td>
    </tr>
    <tr>
      <td>Building Secure and Reliable Systems</td>
      <td>https://sre.google/books/</td>
      <td>Connects reliability and security thinking; especially useful for production architecture reviews.</td>
    </tr>
    <tr>
      <td>Google Reliability Framework</td>
      <td>https://cloud.google.com/architecture/framework/reliability</td>
      <td>Provides a cloud-focused reliability checklist you can use during design and readiness reviews.</td>
    </tr>
    <tr>
      <td>PagerDuty Response Guide</td>
      <td>https://response.pagerduty.com/</td>
      <td>A strong reference for incident command, communications, escalation, and post-incident improvements.</td>
    </tr>
    <tr>
      <td>incident.io Guide</td>
      <td>https://incident.io/guide</td>
      <td>Concise operational advice for running incidents cleanly and communicating clearly under pressure.</td>
    </tr>
    <tr>
      <td>Atlassian Incident Management</td>
      <td>https://www.atlassian.com/incident-management</td>
      <td>Good background on incident lifecycles, postmortems, and stakeholder communication patterns.</td>
    </tr>
    <tr>
      <td>USENIX SREcon</td>
      <td>https://www.usenix.org/srecon</td>
      <td>Conference archive with real-world SRE talks that expose what works beyond textbook theory.</td>
    </tr>
    <tr>
      <td>OpenSLO</td>
      <td>https://github.com/OpenSLO/OpenSLO</td>
      <td>A vendor-neutral SLO specification that helps standardize reliability goals across teams and tools.</td>
    </tr>
    <tr>
      <td>Sloth</td>
      <td>https://sloth.dev/</td>
      <td>Generates Prometheus SLO rules, which reduces manual error when implementing error-budget alerting.</td>
    </tr>
  </tbody>
</table>

## 3. Linux Documentation

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
      <td>man7.org</td>
      <td>https://man7.org/</td>
      <td>One of the best online mirrors for Linux man pages, kernel interfaces, and syscalls.</td>
    </tr>
    <tr>
      <td>kernel.org Documentation</td>
      <td>https://www.kernel.org/doc/html/latest/</td>
      <td>The primary source for kernel internals, admin guides, filesystems, and subsystem documentation.</td>
    </tr>
    <tr>
      <td>Kernel Admin Guide</td>
      <td>https://www.kernel.org/doc/html/latest/admin-guide/</td>
      <td>Useful for boot parameters, sysctl tuning, and production kernel configuration questions.</td>
    </tr>
    <tr>
      <td>Linux From Scratch</td>
      <td>https://www.linuxfromscratch.org/</td>
      <td>Teaches how a Linux system fits together by building one component by component.</td>
    </tr>
    <tr>
      <td>Beyond Linux From Scratch</td>
      <td>https://www.linuxfromscratch.org/blfs/</td>
      <td>Extends LFS with service, desktop, and library guidance, which helps with deeper platform understanding.</td>
    </tr>
    <tr>
      <td>The Linux Documentation Project</td>
      <td>https://tldp.org/</td>
      <td>Contains HOWTOs and classic references that still help with older tools and Unix conventions.</td>
    </tr>
    <tr>
      <td>ArchWiki</td>
      <td>https://wiki.archlinux.org/</td>
      <td>One of the most practical Linux references on the web; excellent for troubleshooting and configuration examples.</td>
    </tr>
    <tr>
      <td>RHEL Documentation</td>
      <td>https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/</td>
      <td>Authoritative for enterprise Linux administration, SELinux, system roles, and supported workflows.</td>
    </tr>
    <tr>
      <td>Ubuntu Server Guide</td>
      <td>https://ubuntu.com/server/docs</td>
      <td>Clear walkthroughs for networking, storage, package management, and common Ubuntu server tasks.</td>
    </tr>
    <tr>
      <td>Debian Administrator's Handbook</td>
      <td>https://debian-handbook.info/</td>
      <td>Strong conceptual and operational reference for Debian packaging, policy, and admin practices.</td>
    </tr>
    <tr>
      <td>Alpine Linux Documentation</td>
      <td>https://docs.alpinelinux.org/</td>
      <td>Useful when working with minimal containers or musl-based systems where behavior differs from glibc distros.</td>
    </tr>
    <tr>
      <td>Gentoo Handbook</td>
      <td>https://wiki.gentoo.org/wiki/Handbook:Main_Page</td>
      <td>Helpful for understanding bootstrapping, kernels, init systems, and package build customization.</td>
    </tr>
    <tr>
      <td>openSUSE Documentation</td>
      <td>https://doc.opensuse.org/</td>
      <td>Good reference for YaST, transactional updates, and SUSE-flavored administration workflows.</td>
    </tr>
    <tr>
      <td>GNU Bash Manual</td>
      <td>https://www.gnu.org/software/bash/manual/</td>
      <td>The definitive shell reference for scripting, expansion rules, quoting, and interactive features.</td>
    </tr>
    <tr>
      <td>GNU Coreutils Manual</td>
      <td>https://www.gnu.org/software/coreutils/manual/</td>
      <td>Explains the flags and edge cases behind everyday commands like cp, mv, sort, and stat.</td>
    </tr>
    <tr>
      <td>systemd Manual Pages</td>
      <td>https://www.freedesktop.org/software/systemd/man/latest/</td>
      <td>Essential when managing services, timers, logging, targets, slices, and unit file behavior.</td>
    </tr>
  </tbody>
</table>

## 4. Security References

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

## 5. Performance

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
      <td>Brendan Gregg</td>
      <td>https://www.brendangregg.com/</td>
      <td>The go-to site for Linux performance engineering, observability, and practical troubleshooting methods.</td>
    </tr>
    <tr>
      <td>USE Method</td>
      <td>https://www.brendangregg.com/usemethod.html</td>
      <td>A fast framework for checking utilization, saturation, and errors across every major resource.</td>
    </tr>
    <tr>
      <td>Flame Graphs</td>
      <td>https://www.brendangregg.com/flamegraphs.html</td>
      <td>Explains one of the most powerful ways to visualize CPU stack samples and hotspots.</td>
    </tr>
    <tr>
      <td>Linux Perf Wiki</td>
      <td>https://perfwiki.github.io/main/</td>
      <td>Good reference for perf tooling, event sampling, and performance counter analysis.</td>
    </tr>
    <tr>
      <td>Phoronix Test Suite</td>
      <td>https://www.phoronix-test-suite.com/</td>
      <td>Useful for reproducible benchmarking and comparative performance testing across systems.</td>
    </tr>
    <tr>
      <td>perf-tools</td>
      <td>https://github.com/brendangregg/perf-tools</td>
      <td>A practical toolbox of scripts for tracing latency, scheduling, and I/O behavior quickly.</td>
    </tr>
    <tr>
      <td>BCC</td>
      <td>https://github.com/iovisor/bcc</td>
      <td>Provides eBPF-based tracing tools that make kernel and application analysis dramatically easier.</td>
    </tr>
    <tr>
      <td>bpftrace</td>
      <td>https://bpftrace.org/</td>
      <td>A concise tracing language for eBPF that speeds up ad hoc performance investigations.</td>
    </tr>
    <tr>
      <td>Performance Co-Pilot Docs</td>
      <td>https://pcp.readthedocs.io/</td>
      <td>Strong for time-series performance telemetry, historical analysis, and long-lived system profiling.</td>
    </tr>
    <tr>
      <td>fio Documentation</td>
      <td>https://fio.readthedocs.io/</td>
      <td>Essential for designing realistic storage and filesystem benchmarks instead of guesswork tests.</td>
    </tr>
    <tr>
      <td>stress-ng</td>
      <td>https://wiki.ubuntu.com/Kernel/Reference/stress-ng</td>
      <td>Useful for safely stressing CPU, memory, and I/O paths to validate resilience and tuning.</td>
    </tr>
    <tr>
      <td>iperf3</td>
      <td>https://software.es.net/iperf/</td>
      <td>The standard network throughput test tool for link validation and performance comparisons.</td>
    </tr>
  </tbody>
</table>

## 6. Container & Kubernetes

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
      <td>Docker Hub</td>
      <td>https://hub.docker.com/</td>
      <td>The central image registry for finding base images, tags, and image metadata quickly.</td>
    </tr>
    <tr>
      <td>Docker Documentation</td>
      <td>https://docs.docker.com/</td>
      <td>Authoritative reference for building, running, securing, and troubleshooting containers.</td>
    </tr>
    <tr>
      <td>Kubernetes Documentation</td>
      <td>https://kubernetes.io/docs/</td>
      <td>The best starting point for concepts, configuration, API references, and operational guidance.</td>
    </tr>
    <tr>
      <td>kubectl Reference</td>
      <td>https://kubernetes.io/docs/reference/kubectl/</td>
      <td>Useful when you need exact kubectl syntax without relying on memory.</td>
    </tr>
    <tr>
      <td>Helm Documentation</td>
      <td>https://helm.sh/docs/</td>
      <td>Explains chart structure, values, templating, and release management patterns.</td>
    </tr>
    <tr>
      <td>Artifact Hub</td>
      <td>https://artifacthub.io/</td>
      <td>The modern discovery portal for Helm charts, operators, and OCI-packaged components.</td>
    </tr>
    <tr>
      <td>OCI Image Spec</td>
      <td>https://github.com/opencontainers/image-spec</td>
      <td>Useful for understanding how container images are structured and transported.</td>
    </tr>
    <tr>
      <td>OCI Runtime Spec</td>
      <td>https://github.com/opencontainers/runtime-spec</td>
      <td>Helps explain low-level container runtime behavior and compatibility expectations.</td>
    </tr>
    <tr>
      <td>Play with Docker</td>
      <td>https://labs.play-with-docker.com/</td>
      <td>A fast sandbox for trying Docker commands without changing your local machine.</td>
    </tr>
    <tr>
      <td>Play with Kubernetes</td>
      <td>https://labs.play-with-k8s.com/</td>
      <td>Useful for quick Kubernetes experiments and demos when you do not want a local cluster.</td>
    </tr>
    <tr>
      <td>kind</td>
      <td>https://kind.sigs.k8s.io/</td>
      <td>Excellent for disposable local clusters used in testing, CI, and troubleshooting workflows.</td>
    </tr>
    <tr>
      <td>Minikube</td>
      <td>https://minikube.sigs.k8s.io/docs/</td>
      <td>Simple local Kubernetes environment that is easy to start with and widely documented.</td>
    </tr>
    <tr>
      <td>Kubernetes the Hard Way</td>
      <td>https://github.com/kelseyhightower/kubernetes-the-hard-way</td>
      <td>Still one of the best ways to understand what cluster components actually do.</td>
    </tr>
    <tr>
      <td>Podman Documentation</td>
      <td>https://docs.podman.io/</td>
      <td>Useful for rootless container workflows and daemonless environments on Linux.</td>
    </tr>
  </tbody>
</table>

## 7. Package Repositories

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
      <td>pkgs.org</td>
      <td>https://pkgs.org/</td>
      <td>Fast package lookup across many Linux distributions when hunting down a specific file or version.</td>
    </tr>
    <tr>
      <td>Repology</td>
      <td>https://repology.org/</td>
      <td>Compares package versions across repositories, making freshness and lag easy to see.</td>
    </tr>
    <tr>
      <td>Flathub</td>
      <td>https://flathub.org/</td>
      <td>The main Flatpak app repository; useful for desktop software discovery across distributions.</td>
    </tr>
    <tr>
      <td>Snapcraft Store</td>
      <td>https://snapcraft.io/store</td>
      <td>Lets you browse snap packages and understand published channels and publisher metadata.</td>
    </tr>
    <tr>
      <td>PyPI</td>
      <td>https://pypi.org/</td>
      <td>The standard Python package index for version checks, docs links, and release history.</td>
    </tr>
    <tr>
      <td>npm</td>
      <td>https://www.npmjs.com/</td>
      <td>The main JavaScript package registry; helpful for dependency research and maintenance checks.</td>
    </tr>
    <tr>
      <td>crates.io</td>
      <td>https://crates.io/</td>
      <td>Rust package registry with versioning, feature flags, and project metadata.</td>
    </tr>
    <tr>
      <td>Maven Central</td>
      <td>https://search.maven.org/</td>
      <td>Useful for Java artifact lookup, coordinates, and release visibility.</td>
    </tr>
    <tr>
      <td>RubyGems</td>
      <td>https://rubygems.org/</td>
      <td>Standard Ruby package source for versions, changelogs, and dependency review.</td>
    </tr>
    <tr>
      <td>NuGet</td>
      <td>https://www.nuget.org/</td>
      <td>Main .NET package registry; helpful during cross-platform build and runtime troubleshooting.</td>
    </tr>
    <tr>
      <td>Arch Linux Packages</td>
      <td>https://archlinux.org/packages/</td>
      <td>Great for finding package ownership, file lists, and version details on Arch-based systems.</td>
    </tr>
    <tr>
      <td>Debian Packages</td>
      <td>https://packages.debian.org/</td>
      <td>Useful when you need package contents, source links, and availability by Debian release.</td>
    </tr>
  </tbody>
</table>

## 8. Learning & Practice

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
      <td>OverTheWire Bandit</td>
      <td>https://overthewire.org/wargames/bandit/</td>
      <td>Great for learning shell basics, SSH usage, file permissions, and Linux problem solving hands-on.</td>
    </tr>
    <tr>
      <td>Linux Journey</td>
      <td>https://linuxjourney.com/</td>
      <td>A beginner-friendly path through Linux concepts with small, digestible lessons.</td>
    </tr>
    <tr>
      <td>sadservers</td>
      <td>https://sadservers.com/</td>
      <td>Practice fixing realistic broken systems, which is excellent prep for interviews and incidents.</td>
    </tr>
    <tr>
      <td>HackerRank Shell</td>
      <td>https://www.hackerrank.com/domains/shell</td>
      <td>Good for sharpening shell scripting under time pressure with structured exercises.</td>
    </tr>
    <tr>
      <td>KodeKloud</td>
      <td>https://kodekloud.com/</td>
      <td>Offers hands-on DevOps, Linux, and Kubernetes labs that build practical operational muscle.</td>
    </tr>
    <tr>
      <td>Linux Survival</td>
      <td>https://linuxsurvival.com/</td>
      <td>A simple interactive shell tutorial that helps new users gain confidence quickly.</td>
    </tr>
    <tr>
      <td>The Missing Semester</td>
      <td>https://missing.csail.mit.edu/</td>
      <td>Teaches the command-line and tooling habits that dramatically improve engineering productivity.</td>
    </tr>
    <tr>
      <td>Killercoda</td>
      <td>https://killercoda.com/</td>
      <td>Provides browser-based Kubernetes and Linux labs with no local setup required.</td>
    </tr>
    <tr>
      <td>freeCodeCamp Linux Commands Handbook</td>
      <td>https://www.freecodecamp.org/news/the-linux-commands-handbook/</td>
      <td>A quick reference for common commands when onboarding new Linux users.</td>
    </tr>
    <tr>
      <td>Red Hat Enable Sysadmin</td>
      <td>https://www.redhat.com/sysadmin</td>
      <td>Readable articles that blend Linux administration fundamentals with modern automation practices.</td>
    </tr>
    <tr>
      <td>DigitalOcean Tutorials</td>
      <td>https://www.digitalocean.com/community/tutorials?primary_filter=linux</td>
      <td>One of the most approachable sources for Linux setup, service deployment, and command examples.</td>
    </tr>
    <tr>
      <td>LabEx Linux Skill Tree</td>
      <td>https://labex.io/skilltrees/linux</td>
      <td>Structured labs for practicing Linux commands, scripting, and administration concepts interactively.</td>
    </tr>
  </tbody>
</table>

## 9. Monitoring & Observability

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
      <td>Prometheus</td>
      <td>https://prometheus.io/</td>
      <td>The core open-source metrics and alerting stack for modern Linux and cloud-native systems.</td>
    </tr>
    <tr>
      <td>Alertmanager</td>
      <td>https://prometheus.io/docs/alerting/latest/alertmanager/</td>
      <td>Explains grouping, routing, deduplication, and silence management for actionable alerting.</td>
    </tr>
    <tr>
      <td>node_exporter</td>
      <td>https://github.com/prometheus/node_exporter</td>
      <td>Essential for exposing host-level metrics such as CPU, memory, filesystem, and network counters.</td>
    </tr>
    <tr>
      <td>blackbox_exporter</td>
      <td>https://github.com/prometheus/blackbox_exporter</td>
      <td>Useful for probing HTTP, TCP, ICMP, and DNS endpoints from the outside in.</td>
    </tr>
    <tr>
      <td>Grafana</td>
      <td>https://grafana.com/</td>
      <td>The standard dashboard and visualization layer for metrics, logs, and traces.</td>
    </tr>
    <tr>
      <td>Loki</td>
      <td>https://grafana.com/oss/loki/</td>
      <td>A label-based log system that pairs well with Prometheus-style operational workflows.</td>
    </tr>
    <tr>
      <td>Jaeger</td>
      <td>https://www.jaegertracing.io/</td>
      <td>Distributed tracing system that helps uncover latency bottlenecks across service boundaries.</td>
    </tr>
    <tr>
      <td>OpenTelemetry</td>
      <td>https://opentelemetry.io/</td>
      <td>The open standard for collecting metrics, logs, and traces without vendor lock-in.</td>
    </tr>
    <tr>
      <td>Elastic Stack</td>
      <td>https://www.elastic.co/elastic-stack</td>
      <td>Widely used for log aggregation, search, visualization, and general observability pipelines.</td>
    </tr>
    <tr>
      <td>Netdata</td>
      <td>https://www.netdata.cloud/</td>
      <td>Excellent for instant, host-centric dashboards with very low setup effort.</td>
    </tr>
    <tr>
      <td>Datadog</td>
      <td>https://www.datadoghq.com/</td>
      <td>Popular commercial observability platform with deep integrations and fast time to value.</td>
    </tr>
    <tr>
      <td>VictoriaMetrics</td>
      <td>https://victoriametrics.com/</td>
      <td>High-performance metrics storage that is especially useful at larger retention scales.</td>
    </tr>
    <tr>
      <td>Thanos</td>
      <td>https://thanos.io/</td>
      <td>Adds long-term storage, deduplication, and global query capabilities to Prometheus deployments.</td>
    </tr>
    <tr>
      <td>Fluent Bit</td>
      <td>https://fluentbit.io/</td>
      <td>A lightweight log and metrics forwarder that fits well on Linux hosts and Kubernetes nodes.</td>
    </tr>
  </tbody>
</table>

## 10. DevOps & Automation

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
      <td>Ansible Galaxy</td>
      <td>https://galaxy.ansible.com/</td>
      <td>Find reusable roles and collections instead of rebuilding automation from scratch.</td>
    </tr>
    <tr>
      <td>Ansible Documentation</td>
      <td>https://docs.ansible.com/</td>
      <td>Authoritative playbook, module, and inventory guidance for configuration management work.</td>
    </tr>
    <tr>
      <td>Terraform Registry</td>
      <td>https://registry.terraform.io/</td>
      <td>The main source for providers and modules used in infrastructure-as-code workflows.</td>
    </tr>
    <tr>
      <td>Terraform Documentation</td>
      <td>https://developer.hashicorp.com/terraform/docs</td>
      <td>Essential for state, plans, modules, and expression syntax when building reliable IaC.</td>
    </tr>
    <tr>
      <td>OpenTofu Documentation</td>
      <td>https://opentofu.org/docs/</td>
      <td>Useful if you prefer an open governance alternative with Terraform-compatible workflows.</td>
    </tr>
    <tr>
      <td>Puppet Forge</td>
      <td>https://forge.puppet.com/</td>
      <td>Finds maintained Puppet modules and common implementation patterns quickly.</td>
    </tr>
    <tr>
      <td>Chef Supermarket</td>
      <td>https://supermarket.chef.io/</td>
      <td>Repository of Chef cookbooks that accelerates bootstrapping and configuration reuse.</td>
    </tr>
    <tr>
      <td>Vagrant Cloud</td>
      <td>https://app.vagrantup.com/</td>
      <td>Useful for browsing box images and building repeatable local lab environments.</td>
    </tr>
    <tr>
      <td>Packer Documentation</td>
      <td>https://developer.hashicorp.com/packer/docs</td>
      <td>Important for golden-image pipelines and reproducible machine image creation.</td>
    </tr>
    <tr>
      <td>GitHub Actions Marketplace</td>
      <td>https://github.com/marketplace?type=actions</td>
      <td>Lets you find reusable CI/CD actions quickly while comparing popularity and maintenance signals.</td>
    </tr>
    <tr>
      <td>Jenkins Documentation</td>
      <td>https://www.jenkins.io/doc/</td>
      <td>Still valuable in environments with established Jenkins pipelines and plugin ecosystems.</td>
    </tr>
    <tr>
      <td>Argo CD</td>
      <td>https://argo-cd.readthedocs.io/</td>
      <td>Key GitOps reference for syncing Kubernetes state from version control safely.</td>
    </tr>
  </tbody>
</table>

## 11. Cheat Sheets & Quick References

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
      <td>cheat.sh</td>
      <td>https://cheat.sh/</td>
      <td>Fast command-line cheat sheet service; great for quick examples like curl cheat.sh/tar.</td>
    </tr>
    <tr>
      <td>tldr pages</td>
      <td>https://tldr.sh/</td>
      <td>Short, example-driven command references that are easier to scan than full man pages.</td>
    </tr>
    <tr>
      <td>explainshell</td>
      <td>https://explainshell.com/</td>
      <td>Breaks down shell commands flag by flag, which is excellent for learning unfamiliar pipelines.</td>
    </tr>
    <tr>
      <td>crontab.guru</td>
      <td>https://crontab.guru/</td>
      <td>Quickly decodes cron expressions so you can validate schedules before production changes.</td>
    </tr>
    <tr>
      <td>regex101</td>
      <td>https://regex101.com/</td>
      <td>Excellent for debugging regular expressions with explanations, tests, and match visualization.</td>
    </tr>
    <tr>
      <td>jq play</td>
      <td>https://jqplay.org/</td>
      <td>Useful for iterating on jq filters interactively before embedding them into scripts.</td>
    </tr>
    <tr>
      <td>ShellCheck</td>
      <td>https://www.shellcheck.net/</td>
      <td>Finds common shell scripting mistakes and unsafe patterns before they become outages.</td>
    </tr>
    <tr>
      <td>chmod-calculator.com</td>
      <td>https://chmod-calculator.com/</td>
      <td>Helpful when translating symbolic permissions into octal values quickly and safely.</td>
    </tr>
    <tr>
      <td>Devhints</td>
      <td>https://devhints.io/</td>
      <td>Compact cheat sheets across many tools, languages, and CLIs used by Linux engineers.</td>
    </tr>
    <tr>
      <td>QuickRef.ME</td>
      <td>https://quickref.me/</td>
      <td>A broad collection of concise syntax references that save time during context switching.</td>
    </tr>
    <tr>
      <td>SS64</td>
      <td>https://ss64.com/</td>
      <td>Useful for command syntax lookups and quick reminders across Unix and Windows shells.</td>
    </tr>
    <tr>
      <td>Vim Cheat Sheet</td>
      <td>https://vim.rtorr.com/</td>
      <td>A classic printable Vim quick reference that speeds up navigation and editing recall.</td>
    </tr>
    <tr>
      <td>tmux Cheat Sheet</td>
      <td>https://tmuxcheatsheet.com/</td>
      <td>Good for remembering panes, sessions, copy mode, and multiplexing shortcuts.</td>
    </tr>
    <tr>
      <td>Sed One-Liners</td>
      <td>https://www.grymoire.com/Unix/Sed.html</td>
      <td>Very useful when you need compact text-processing tricks for logs and config files.</td>
    </tr>
  </tbody>
</table>

## 12. Essential RFCs Table

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

## 13. System Administration References

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
      <td>Kernel Parameters</td>
      <td>https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html</td>
      <td>Essential for boot-time debugging and understanding kernel command-line behavior.</td>
    </tr>
    <tr>
      <td>systemd.io</td>
      <td>https://systemd.io/</td>
      <td>High-level systemd design and reference material that complements the man pages nicely.</td>
    </tr>
    <tr>
      <td>systemd Manual Pages</td>
      <td>https://www.freedesktop.org/software/systemd/man/latest/</td>
      <td>Best source for units, journald, timers, slices, targets, and dependency semantics.</td>
    </tr>
    <tr>
      <td>Filesystem Hierarchy Standard</td>
      <td>https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html</td>
      <td>Explains why files belong where they do, which helps with packaging and policy decisions.</td>
    </tr>
    <tr>
      <td>POSIX Specification</td>
      <td>https://pubs.opengroup.org/onlinepubs/9699919799/</td>
      <td>Useful when writing portable shell or C code that must work beyond one Linux distro.</td>
    </tr>
    <tr>
      <td>procfs Documentation</td>
      <td>https://www.kernel.org/doc/html/latest/filesystems/proc.html</td>
      <td>Good reference for interpreting /proc data during host-level debugging and introspection.</td>
    </tr>
    <tr>
      <td>sysfs Documentation</td>
      <td>https://www.kernel.org/doc/html/latest/filesystems/sysfs.html</td>
      <td>Explains the device and kernel attribute model exposed under /sys.</td>
    </tr>
    <tr>
      <td>sysctl Explorer</td>
      <td>https://sysctl-explorer.net/</td>
      <td>Convenient lookup tool for kernel tunables when you need quick context on sysctl keys.</td>
    </tr>
    <tr>
      <td>XDG Base Directory Specification</td>
      <td>https://specifications.freedesktop.org/basedir-spec/latest/</td>
      <td>Helpful when reasoning about user config, cache, and data file locations.</td>
    </tr>
    <tr>
      <td>sudo Documentation</td>
      <td>https://www.sudo.ws/docs/</td>
      <td>Useful for delegation, policy, logging, and command restriction design.</td>
    </tr>
    <tr>
      <td>Linux-PAM Documentation</td>
      <td>https://www.linux-pam.org/Linux-PAM-html/</td>
      <td>Important when troubleshooting authentication stacks and account policy behavior.</td>
    </tr>
    <tr>
      <td>nftables Wiki</td>
      <td>https://wiki.nftables.org/</td>
      <td>Great reference for modern Linux firewall rulesets and packet filtering patterns.</td>
    </tr>
  </tbody>
</table>

## 14. Cloud Provider CLIs & Docs

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
      <td>AWS CLI Command Reference</td>
      <td>https://docs.aws.amazon.com/cli/latest/reference/</td>
      <td>The fastest way to find exact aws subcommands, options, and output shapes.</td>
    </tr>
    <tr>
      <td>AWS CLI User Guide</td>
      <td>https://docs.aws.amazon.com/cli/latest/userguide/</td>
      <td>Useful for profiles, credentials, config files, and common automation patterns.</td>
    </tr>
    <tr>
      <td>gcloud CLI Reference</td>
      <td>https://cloud.google.com/sdk/gcloud/reference</td>
      <td>Command-level documentation for Google Cloud operations from the terminal.</td>
    </tr>
    <tr>
      <td>Google Cloud CLI Docs</td>
      <td>https://cloud.google.com/sdk/docs</td>
      <td>Covers installation, authentication, components, and broader CLI workflow guidance.</td>
    </tr>
    <tr>
      <td>Azure CLI Reference</td>
      <td>https://learn.microsoft.com/cli/azure/reference-index</td>
      <td>Best source for exact az syntax and service-specific command groups.</td>
    </tr>
    <tr>
      <td>Azure CLI Docs</td>
      <td>https://learn.microsoft.com/cli/azure/</td>
      <td>Good starting point for installation, login models, scripting, and output control.</td>
    </tr>
    <tr>
      <td>DigitalOcean doctl Docs</td>
      <td>https://docs.digitalocean.com/reference/doctl/</td>
      <td>Useful if you automate droplets, load balancers, and managed services from Linux hosts.</td>
    </tr>
    <tr>
      <td>Linode CLI Docs</td>
      <td>https://www.linode.com/docs/products/tools/cli/</td>
      <td>Helpful reference for Linode environments where terminal-first workflows are preferred.</td>
    </tr>
    <tr>
      <td>Oracle OCI CLI Docs</td>
      <td>https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cli.htm</td>
      <td>Useful for OCI automation, profile management, and scripting infrastructure tasks.</td>
    </tr>
    <tr>
      <td>Alibaba Cloud CLI Docs</td>
      <td>https://www.alibabacloud.com/help/en/cli/</td>
      <td>Good reference when operating Linux workloads across Alibaba Cloud services.</td>
    </tr>
  </tbody>
</table>

## 15. Online Tools

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
      <td>JSONLint</td>
      <td>https://jsonlint.com/</td>
      <td>Quickly validates JSON syntax when APIs or config files refuse to parse.</td>
    </tr>
    <tr>
      <td>JSON Formatter &amp; Validator</td>
      <td>https://jsonformatter.org/</td>
      <td>Useful for prettifying and validating dense JSON payloads during debugging.</td>
    </tr>
    <tr>
      <td>YAML Lint</td>
      <td>https://www.yamllint.com/</td>
      <td>Handy for checking indentation and syntax before applying YAML to production systems.</td>
    </tr>
    <tr>
      <td>JSONPath Online Evaluator</td>
      <td>https://jsonpath.com/</td>
      <td>Lets you test JSONPath expressions interactively before using them in automation.</td>
    </tr>
    <tr>
      <td>Base64 Guru</td>
      <td>https://base64.guru/</td>
      <td>Useful for encoding, decoding, and inspecting base64 blobs from logs and APIs.</td>
    </tr>
    <tr>
      <td>CyberChef</td>
      <td>https://gchq.github.io/CyberChef/</td>
      <td>A powerful browser-side toolbox for transforms, encodings, hashes, and data cleanup.</td>
    </tr>
    <tr>
      <td>Epoch Converter</td>
      <td>https://www.epochconverter.com/</td>
      <td>Very useful when translating Unix timestamps during incident timelines and log review.</td>
    </tr>
    <tr>
      <td>unixtimestamp.com</td>
      <td>https://www.unixtimestamp.com/</td>
      <td>Another quick timestamp conversion tool that is handy to keep bookmarked.</td>
    </tr>
    <tr>
      <td>JWT.io</td>
      <td>https://jwt.io/</td>
      <td>Useful for decoding JWT structure and understanding claims during auth debugging.</td>
    </tr>
    <tr>
      <td>URLEncoder</td>
      <td>https://www.urlencoder.org/</td>
      <td>Fast encode and decode tool for query strings, form data, and shell-safe URLs.</td>
    </tr>
    <tr>
      <td>DNS Checker</td>
      <td>https://dnschecker.org/</td>
      <td>Helps verify global DNS propagation across resolvers when changes appear inconsistent.</td>
    </tr>
    <tr>
      <td>SSL Labs Server Test</td>
      <td>https://www.ssllabs.com/ssltest/</td>
      <td>Useful for public TLS verification, certificate chain checks, and protocol hardening review.</td>
    </tr>
    <tr>
      <td>Diffchecker</td>
      <td>https://www.diffchecker.com/</td>
      <td>Convenient for quickly comparing config snippets, command output, or generated files.</td>
    </tr>
    <tr>
      <td>UUID Generator</td>
      <td>https://www.uuidgenerator.net/</td>
      <td>Handy when you need a quick UUID during testing, templating, or example generation.</td>
    </tr>
  </tbody>
</table>

---

## How to use this guide effectively

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
      <td>Bookmark strategy</td>
      <td>N/A</td>
      <td>Keep the top 10 references in your browser bar and the rest in organized folders by topic.</td>
    </tr>
    <tr>
      <td>CLI-first habit</td>
      <td>N/A</td>
      <td>Prefer refs with exact command examples when you are on-call and need answers fast.</td>
    </tr>
    <tr>
      <td>Cross-check practice</td>
      <td>N/A</td>
      <td>Use official docs plus community references together to avoid outdated or incomplete guidance.</td>
    </tr>
    <tr>
      <td>Protocol deep dives</td>
      <td>N/A</td>
      <td>When behavior seems strange, go to the RFC rather than relying on blog summaries.</td>
    </tr>
    <tr>
      <td>Learning loop</td>
      <td>N/A</td>
      <td>Read a concept, try it in a lab, then document the command sequence you used successfully.</td>
    </tr>
  </tbody>
</table>
