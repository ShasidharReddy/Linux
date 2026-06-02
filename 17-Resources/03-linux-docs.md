# Linux Documentation

This guide collects Linux documentation, system administration references, distro material, and cloud CLI docs.

## Linux Documentation

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


## System Administration References

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


## Cloud Provider CLIs & Docs

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
