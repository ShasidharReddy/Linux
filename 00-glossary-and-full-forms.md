# 00. Glossary and Full Forms

This glossary centralizes the abbreviations, full forms, and short explanations used across the repository.
Use it as the master reference when a chapter introduces a tool, protocol, security term, or infrastructure concept.

## How to use this glossary

- Check here when a short form appears before it is fully explained in a chapter.
- Prefer the chapter-specific **Full Forms & Terminology** or **Key Terms** boxes for quick context.
- Return here for the longer cross-category reference.

## Commands and tools

| Abbreviation | Full Form / Meaning | Brief Explanation |
| --- | --- | --- |
| vi | Visual Interface / Visual Editor | Classic text editor created by Bill Joy in 1976. |
| vim | Vi IMproved | Enhanced version of vi created by Bram Moolenaar. |
| ss | Socket Statistics | Modern replacement for netstat that shows socket details. |
| netstat | Network Statistics | Legacy tool for showing network connections and routes. |
| grep | Global Regular Expression Print | Searches text using literal strings or patterns. |
| awk | Aho, Weinberger, and Kernighan | Text processing language named after its creators. |
| sed | Stream Editor | Non-interactive editor for transforming text streams. |
| cat | Concatenate | Reads and prints file content to standard output. |
| ls | List | Lists directory contents. |
| cd | Change Directory | Changes the current working directory. |
| pwd | Print Working Directory | Shows the full path of the current directory. |
| cp | Copy | Copies files and directories. |
| mv | Move | Moves or renames files and directories. |
| rm | Remove | Deletes files and directories. |
| mkdir | Make Directory | Creates new directories. |
| chmod | Change Mode | Changes file permission bits. |
| chown | Change Owner | Changes file ownership. |
| chgrp | Change Group | Changes file group ownership. |
| df | Disk Free | Shows filesystem space usage. |
| du | Disk Usage | Shows how much space files or directories use. |
| ps | Process Status | Lists running processes. |
| top | Table of Processes | Real-time process monitor. |
| htop | Hisham's Top | Interactive process viewer by Hisham Muhammad. |
| kill | Kill | Sends a signal to a process. |
| sudo | Super User Do | Runs a command as another user, commonly root. |
| su | Switch User / Substitute User | Starts a shell as another user. |
| ssh | Secure Shell | Encrypted remote access protocol and client command. |
| scp | Secure Copy Protocol | Copies files over SSH. |
| sftp | SSH File Transfer Protocol | Interactive file transfer over SSH. |
| rsync | Remote Sync | Efficient file synchronization tool. |
| tar | Tape Archive | Archives many files into a single stream or file. |
| gzip | GNU Zip | Compression utility commonly paired with tar. |
| curl | Client URL | Transfers data to or from URLs. |
| wget | Web Get | Downloads files from web or FTP endpoints. |
| apt | Advanced Package Tool | High-level package manager for Debian and Ubuntu. |
| yum | Yellowdog Updater Modified | Legacy RHEL-family package manager. |
| dnf | Dandified YUM | Modern YUM replacement on RHEL-family systems. |
| rpm | Red Hat Package Manager | Low-level package format and management tool. |
| dpkg | Debian Package | Low-level package tool for Debian-based systems. |
| systemctl | System Control | Manages systemd units and services. |
| journalctl | Journal Control | Reads logs from the systemd journal. |
| ip | Internet Protocol (tool) | Modern network configuration tool from iproute2. |
| ifconfig | Interface Configuration | Legacy network interface configuration tool. |
| iptables | IP Tables | Legacy Linux firewall management interface. |
| nft | Netfilter Tables | Modern firewall framework and command-line tool. |
| lsblk | List Block Devices | Shows disks, partitions, and block-device layout. |
| fdisk | Format Disk / Fixed Disk | Partition management tool for disks. |
| mkfs | Make Filesystem | Creates a filesystem on a block device. |
| fsck | Filesystem Check | Checks and repairs filesystems. |
| lvm | Logical Volume Manager | Flexible storage management framework. |
| pv | Physical Volume | LVM storage layer backed by a disk or partition. |
| vg | Volume Group | LVM pool built from one or more physical volumes. |
| lv | Logical Volume | Resizable storage volume carved from a volume group. |
| nmcli | NetworkManager CLI | Command-line interface for NetworkManager. |
| tcpdump | TCP Dump | Captures and displays network packets. |
| nmap | Network Mapper | Network discovery and security auditing tool. |
| strace | System Trace | Traces system calls and signals. |
| lsof | List Open Files | Shows open files, sockets, and device handles. |

## Network protocols and identity terms

| Abbreviation | Full Form / Meaning | Brief Explanation |
| --- | --- | --- |
| HTTP | HyperText Transfer Protocol | Transfers web pages, APIs, and other web resources. |
| HTTPS | HyperText Transfer Protocol Secure | HTTP protected by TLS encryption. |
| SSH | Secure Shell | Encrypted remote shell, command, and tunneling protocol. |
| DNS | Domain Name System | Translates names such as example.com into IP addresses. |
| DHCP | Dynamic Host Configuration Protocol | Automatically assigns IP settings to clients. |
| NFS | Network File System | Shares remote directories as mounted filesystems. |
| SMTP | Simple Mail Transfer Protocol | Sends and relays email. |
| IMAP | Internet Message Access Protocol | Retrieves mail while keeping server-side state. |
| POP3 | Post Office Protocol v3 | Downloads mail in a simpler, client-side model. |
| FTP | File Transfer Protocol | Legacy, unencrypted file transfer protocol. |
| FTPS | FTP Secure (FTP over TLS) | FTP wrapped with TLS encryption. |
| SFTP | SSH File Transfer Protocol | Secure file transfer subsystem carried over SSH. |
| SCP | Secure Copy Protocol | Simple file copy mechanism over SSH. |
| LDAP | Lightweight Directory Access Protocol | Directory lookup and identity service protocol. |
| LDAPS | LDAP over SSL/TLS | LDAP protected by implicit TLS. |
| SNMP | Simple Network Management Protocol | Device monitoring and management protocol. |
| TCP | Transmission Control Protocol | Reliable, ordered transport protocol. |
| UDP | User Datagram Protocol | Lightweight, connectionless transport protocol. |
| TLS | Transport Layer Security | Modern encryption protocol for data in transit. |
| SSL | Secure Sockets Layer | Deprecated predecessor to TLS. |
| ICMP | Internet Control Message Protocol | Supports ping, errors, and control messages. |
| ARP | Address Resolution Protocol | Maps IPv4 addresses to MAC addresses on a LAN. |
| NAT | Network Address Translation | Maps private and public IP addresses. |
| VLAN | Virtual Local Area Network | Creates logical Layer 2 network segments. |
| BGP | Border Gateway Protocol | Inter-domain routing protocol for the Internet. |
| OSPF | Open Shortest Path First | Link-state routing protocol for internal networks. |
| RIP | Routing Information Protocol | Older distance-vector routing protocol. |
| NTP | Network Time Protocol | Synchronizes system clocks. |
| RADIUS | Remote Authentication Dial-In User Service | AAA protocol used for network access control. |
| TACACS+ | Terminal Access Controller Access-Control System Plus | Cisco-oriented AAA protocol. |
| SAML | Security Assertion Markup Language | Federated sign-on protocol for exchanging identity assertions. |
| OAuth | Open Authorization | Delegated authorization framework. |
| OIDC | OpenID Connect | Identity layer built on OAuth 2.0. |
| MFA | Multi-Factor Authentication | Requires more than one authentication factor. |
| TOTP | Time-Based One-Time Password | Time-based one-time code used for 2FA. |
| CIDR | Classless Inter-Domain Routing | Compact network notation such as 192.168.1.0/24. |
| DORA | Discover, Offer, Request, Acknowledge | Four-message DHCP lease workflow. |

## DNS record types and lookup terms

| Record / Term | Full Form / Meaning | Purpose |
| --- | --- | --- |
| A | Address | Maps a domain name to an IPv4 address. |
| AAAA | IPv6 Address | Maps a domain name to an IPv6 address. |
| CNAME | Canonical Name | Creates an alias to another hostname. |
| MX | Mail Exchange | Declares the mail server for a domain. |
| TXT | Text | Stores arbitrary text such as SPF, DKIM, or verification data. |
| NS | Name Server | Declares an authoritative nameserver for a zone. |
| SOA | Start of Authority | Holds zone metadata such as the serial and timers. |
| PTR | Pointer | Maps an IP address back to a hostname. |
| SRV | Service | Locates a service host and port. |
| CAA | Certification Authority Authorization | States which certificate authorities may issue certs. |
| SPF | Sender Policy Framework | Lists approved email senders for a domain. |
| DKIM | DomainKeys Identified Mail | Uses DNS-published keys to verify signed email. |
| DMARC | Domain-based Message Authentication, Reporting & Conformance | Publishes email authentication policy and reporting rules. |
| TTL | Time To Live | Controls how long DNS responses may be cached. |
| AXFR | Authoritative Transfer | Full DNS zone transfer. |
| IXFR | Incremental Zone Transfer | Transfers only changed DNS records. |

## Cryptography and security terms

| Abbreviation | Full Form / Meaning | Brief Explanation |
| --- | --- | --- |
| ed25519 | Edwards-curve Digital Signature Algorithm using Curve25519 | Fast, modern signature system commonly recommended for SSH keys. |
| RSA | Rivest–Shamir–Adleman | Public-key cryptosystem named after its creators. |
| ECDSA | Elliptic Curve Digital Signature Algorithm | Elliptic-curve variant of DSA with smaller keys. |
| DSA | Digital Signature Algorithm | Older signing algorithm that is deprecated for SSH. |
| AES | Advanced Encryption Standard | Modern symmetric encryption standard. |
| DES | Data Encryption Standard | Older 56-bit cipher that is now insecure. |
| SHA | Secure Hash Algorithm | Hash family that includes SHA-256 and SHA-512. |
| MD5 | Message-Digest Algorithm 5 | Broken legacy hash that should not be used for security. |
| GPG | GNU Privacy Guard | Open-source implementation of the PGP standard. |
| PGP | Pretty Good Privacy | Encryption standard for files and email. |
| PKI | Public Key Infrastructure | Processes and trust systems for managing certificates. |
| CA | Certificate Authority | Organization or service that issues digital certificates. |
| CSR | Certificate Signing Request | Request sent to a CA to obtain a certificate. |
| CRL | Certificate Revocation List | Published list of certificates that are no longer trusted. |
| OCSP | Online Certificate Status Protocol | Checks certificate revocation status in real time. |
| HMAC | Hash-based Message Authentication Code | Protects integrity and authenticity using a shared secret. |
| LUKS | Linux Unified Key Setup | Standard format for Linux disk encryption. |
| SELinux | Security-Enhanced Linux | Mandatory access control system originally developed with NSA involvement. |
| AppArmor | Application Armor | Profile-based mandatory access control system. |
| CIS | Center for Internet Security | Publishes hardening benchmarks and controls. |
| CVE | Common Vulnerabilities and Exposures | Standard identifier for a publicly tracked vulnerability. |
| CVSS | Common Vulnerability Scoring System | Numeric scoring framework for vulnerability severity. |

## Infrastructure, cloud, and operations terms

| Abbreviation | Full Form / Meaning | Brief Explanation |
| --- | --- | --- |
| VM | Virtual Machine | Software-defined computer running on a hypervisor. |
| KVM | Kernel-based Virtual Machine | Linux hypervisor built into the kernel. |
| QEMU | Quick Emulator | Machine emulator and virtualizer. |
| LXC | Linux Containers | Operating-system-level container technology. |
| OCI | Open Container Initiative | Container image and runtime standards body. |
| CRI | Container Runtime Interface | Kubernetes API for container runtimes. |
| CNI | Container Network Interface | Specification for container networking plugins. |
| CSI | Container Storage Interface | Specification for container storage plugins. |
| HPA | Horizontal Pod Autoscaler | Kubernetes component that scales pods horizontally. |
| PV | Persistent Volume | Kubernetes storage resource or LVM physical volume depending on context. |
| PVC | Persistent Volume Claim | Kubernetes request for persistent storage. |
| RBAC | Role-Based Access Control | Permission model based on roles and bindings. |
| IPMI | Intelligent Platform Management Interface | Out-of-band server management standard. |
| iLO | Integrated Lights-Out | HPE server remote management platform. |
| iDRAC | Integrated Dell Remote Access Controller | Dell server remote management platform. |
| BMC | Baseboard Management Controller | Dedicated chip for hardware management. |
| PDU | Power Distribution Unit | Rack-level power distribution hardware. |
| UPS | Uninterruptible Power Supply | Battery-backed power protection device. |
| LACP | Link Aggregation Control Protocol | Standard for bundling multiple network links. |
| MLAG | Multi-Chassis Link Aggregation | Link aggregation spanning multiple switches. |
| STP | Spanning Tree Protocol | Prevents Layer 2 loops. |
| RSTP | Rapid Spanning Tree Protocol | Faster-converging STP variant. |
| MTU | Maximum Transmission Unit | Largest packet size a link can carry. |
| IOPS | Input/Output Operations Per Second | Common metric for storage performance. |
| NAS | Network Attached Storage | File-level shared storage accessed over a network. |
| SAN | Storage Area Network | Block-level shared storage network. |
| iSCSI | Internet Small Computer Systems Interface | Carries block-storage traffic over IP. |
| RAID | Redundant Array of Independent Disks | Combines disks for resilience or performance. |
| LUN | Logical Unit Number | Identifier for a storage volume presented by an array. |
| DRBD | Distributed Replicated Block Device | Replicates block devices over the network. |
| HA | High Availability | Design goal of reducing downtime. |
| DR | Disaster Recovery | Plans and systems for recovering after major failure. |
| RTO | Recovery Time Objective | Maximum acceptable service outage time. |
| RPO | Recovery Point Objective | Maximum acceptable data loss window. |
| SLA | Service Level Agreement | Formal uptime or performance commitment. |
| SLO | Service Level Objective | Internal target that supports an SLA. |
| SLI | Service Level Indicator | Measured metric used to evaluate an SLO. |
| MTTR | Mean Time To Recovery | Average time needed to restore service. |
| MTTA | Mean Time To Acknowledge | Average time to acknowledge an incident. |
| MTTF | Mean Time To Failure | Average operating time before a component fails. |
| TCO | Total Cost of Ownership | Full lifecycle cost of a system or platform. |
| CapEx | Capital Expenditure | Upfront purchase spending. |
| OpEx | Operational Expenditure | Ongoing operating expense. |
| IaC | Infrastructure as Code | Managing infrastructure through version-controlled code. |
| CI/CD | Continuous Integration / Continuous Delivery | Automated build, test, and deployment workflow. |
| GitOps | Git Operations | Using Git as the source of truth for operations changes. |
| SRE | Site Reliability Engineering | Operations discipline focused on reliability and automation. |

## Supporting Linux and directory terms

| Abbreviation | Full Form / Meaning | Brief Explanation |
| --- | --- | --- |
| PAM | Pluggable Authentication Modules | Linux framework for authentication policy and modules. |
| NSS | Name Service Switch | Controls how Linux resolves users, groups, and hosts. |
| SSSD | System Security Services Daemon | Caches and brokers identity lookups and auth to external directories. |
| MIB | Management Information Base | Schema used to name SNMP values. |
| OID | Object Identifier | Numeric path used to identify an SNMP object. |
| LDIF | LDAP Data Interchange Format | Text format for importing and exporting LDAP entries. |
| DN | Distinguished Name | Full unique LDAP path for an entry. |
