# Ansible Modules and Plugins

← Back to [12-ansible-deep-dive.md](./12-ansible-deep-dive.md)

Ad-hoc commands, modules, collections, custom modules, and plugin development.

---

## ⚡ 3. Ad-Hoc Commands

### 🧰 What ad-hoc commands are for

Ad-hoc commands are single-command operations executed without creating a playbook.

They are ideal for quick inspections, emergency fixes, one-off service actions, and fleet-wide information gathering.

### 📦 Module categories and common modules

| Category | Examples | Typical use |
|---|---|---|
| Package management | `package`, `apt`, `dnf`, `yum`, `pip` | Install, remove, and update software |
| Service control | `service`, `systemd` | Start, stop, restart, enable services |
| File management | `file`, `copy`, `template`, `lineinfile`, `blockinfile` | Manage files, directories, templates, and config edits |
| Users and access | `user`, `group`, `authorized_key` | Create accounts and manage SSH access |
| Commands | `command`, `shell`, `raw`, `script` | Execute commands when no better module exists |
| Facts and debugging | `setup`, `debug`, `assert` | Inspect state and validate assumptions |
| Networking and security | `firewalld`, `ufw`, `seboolean`, `selinux` | Manage security posture |
| Cloud and APIs | `amazon.aws.*`, `azure.azcollection.*`, `google.cloud.*` | Provision and manage cloud resources |

### 🧪 Ad-hoc command anatomy

```bash
ansible <pattern> -i <inventory> [options] -m <module> -a '<module arguments>'
```

Common options include `-b` for become, `-u` for SSH user, `-f` for forks, `-l` for limit, and `-e` for extra vars.

### 🔐 Privilege escalation

```bash
ansible web -i inventories/prod/hosts.yml -b -m package -a 'name=nginx state=present'
ansible db -i inventories/prod/hosts.yml -b --become-user postgres -m shell -a 'psql -c "SELECT version();"'
```

Set `become: true` in playbooks for repeatable privilege escalation and reserve CLI flags for quick operational tasks.

### 🚦 Parallelism and forks

Ad-hoc commands run in parallel based on the `forks` setting or the `-f` flag.

```bash
ansible all -i inventories/prod/hosts.yml -f 50 -m ping
```

More forks improve speed only if the control node, network, and targets can handle the concurrency.

For very large fleets, combine forks with `--limit` or inventory partitions to avoid overwhelming bastion hosts and package repositories.

### 🧾 26 ad-hoc command examples

#### Example 01: Ping all managed nodes

```bash
ansible all -i inventories/prod/hosts.yml -m ping
```

- Why it matters: Connectivity verification and Python availability check.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 02: Gather uptime from all nodes

```bash
ansible all -i inventories/prod/hosts.yml -a 'uptime'
```

- Why it matters: Fast health check across the fleet.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 03: Check free disk space

```bash
ansible all -i inventories/prod/hosts.yml -a 'df -h /'
```

- Why it matters: Useful before deployments and patching.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 04: Gather memory summary

```bash
ansible all -i inventories/prod/hosts.yml -a 'free -m'
```

- Why it matters: Spot memory pressure quickly.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 05: Collect only hostname facts

```bash
ansible all -i inventories/prod/hosts.yml -m setup -a 'filter=ansible_hostname'
```

- Why it matters: Lighter than full fact gathering.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 06: Install nginx on web nodes

```bash
ansible web -i inventories/prod/hosts.yml -b -m package -a 'name=nginx state=present'
```

- Why it matters: Prefer module-driven package management.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 07: Restart nginx service

```bash
ansible web -i inventories/prod/hosts.yml -b -m service -a 'name=nginx state=restarted'
```

- Why it matters: Idempotent service action.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 08: Enable and start chronyd

```bash
ansible all -i inventories/prod/hosts.yml -b -m systemd -a 'name=chronyd state=started enabled=true'
```

- Why it matters: Good baseline control pattern.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 09: Create an operations user

```bash
ansible all -i inventories/prod/hosts.yml -b -m user -a 'name=opsadmin shell=/bin/bash groups=wheel append=true state=present'
```

- Why it matters: Safe user management without shell scripting.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 10: Push an SSH key

```bash
ansible all -i inventories/prod/hosts.yml -b -m authorized_key -a 'user=opsadmin key=https://github.com/example.keys state=present'
```

- Why it matters: Bootstrap fleet access quickly.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 11: Ensure a directory exists

```bash
ansible all -i inventories/prod/hosts.yml -b -m file -a 'path=/opt/app state=directory owner=root group=root mode=0755'
```

- Why it matters: Common for application layout preparation.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 12: Remove a stale file

```bash
ansible all -i inventories/prod/hosts.yml -b -m file -a 'path=/opt/support/old-release.tgz state=absent'
```

- Why it matters: Cleanup action across many hosts.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 13: Copy a static config file

```bash
ansible web -i inventories/prod/hosts.yml -b -m copy -a 'src=files/nginx.conf dest=/etc/nginx/nginx.conf owner=root group=root mode=0644'
```

- Why it matters: Works when templating is not required.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 14: Patch a single line in a config

```bash
ansible all -i inventories/prod/hosts.yml -b -m lineinfile -a 'path=/etc/ssh/sshd_config regexp=^PermitRootLogin line=PermitRootLogin no'
```

- Why it matters: One-line remediations are a classic `lineinfile` use case.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 15: Insert a managed config block

```bash
ansible all -i inventories/prod/hosts.yml -b -m blockinfile -a 'path=/etc/motd block=Managed\ by\ Ansible'
```

- Why it matters: Clearly marks automation-managed text.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 16: Fetch a file from remote hosts

```bash
ansible db -i inventories/prod/hosts.yml -b -m fetch -a 'src=/etc/my.cnf dest=artifacts/ flat=false'
```

- Why it matters: Useful for audits and incident response.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 17: Run a shell pipeline

```bash
ansible all -i inventories/prod/hosts.yml -b -m shell -a 'journalctl -u nginx --since today | tail -n 20'
```

- Why it matters: Use `shell` only when shell features are actually needed.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 18: Execute a non-shell command safely

```bash
ansible all -i inventories/prod/hosts.yml -m command -a 'cat /etc/os-release'
```

- Why it matters: Preferred over `shell` for simple commands.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 19: Use raw to bootstrap Python

```bash
ansible new_hosts -i inventories/bootstrap.ini -m raw -a 'dnf install -y python3'
```

- Why it matters: Helpful on minimal images before normal modules can run.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 20: Reboot a patch window batch

```bash
ansible app -i inventories/prod/hosts.yml -b -m reboot -a 'reboot_timeout=900 test_command=whoami'
```

- Why it matters: Handles disconnect and reconnection cleanly.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 21: Mount a filesystem

```bash
ansible storage -i inventories/prod/hosts.yml -b -m mount -a 'path=/data src=/dev/vdb fstype=xfs state=mounted'
```

- Why it matters: Persistent mount management.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 22: Toggle firewalld service rule

```bash
ansible web -i inventories/prod/hosts.yml -b -m ansible.posix.firewalld -a 'service=http permanent=true state=enabled immediate=true'
```

- Why it matters: Service-oriented firewall management.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 23: Archive logs on remote hosts

```bash
ansible all -i inventories/prod/hosts.yml -b -m community.general.archive -a 'path=/var/log/nginx dest=/root/nginx-logs.tgz format=gz'
```

- Why it matters: Fast evidence capture.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 24: Schedule a cron job

```bash
ansible backup -i inventories/prod/hosts.yml -b -m cron -a 'name=db-backup minute=0 hour=2 user=root job=/usr/local/bin/db-backup.sh'
```

- Why it matters: Ad-hoc cron creation when formal playbooks are not yet available.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 25: Display a variable for a group

```bash
ansible web -i inventories/prod/hosts.yml -m debug -a 'var=hostvars[inventory_hostname].http_port'
```

- Why it matters: Inspect resolved inventory values.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

#### Example 26: Run with high parallelism

```bash
ansible all -i inventories/prod/hosts.yml -f 100 -m ping
```

- Why it matters: Stress-test connectivity or accelerate a broad read-only task.

- Operational tip: verify inventory targeting before destructive actions and add `--limit` for safety during emergency runs.

## 🌌 7. Ansible Galaxy & Collections

### 🧱 Galaxy roles

```yaml
# requirements.yml
roles:
  - name: geerlingguy.nginx
    version: 3.2.0
collections:
  - name: ansible.posix
    version: 1.5.4
  - name: community.general
    version: 9.1.0
```

```bash
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install -r requirements.yml
```

### 📦 Collections concept

Collections are the modern packaging unit for modules, plugins, roles, playbooks, and documentation.

Fully qualified collection names such as `ansible.builtin.copy` or `community.general.timezone` reduce ambiguity and make dependencies explicit.

### ⬇️ Installing and using collections

```bash
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install amazon.aws:==7.6.1
```

```yaml
---
- name: Use collection-backed modules
  hosts: web
  become: true
  collections:
    - ansible.posix
    - community.general
  tasks:
    - name: Open firewall rule
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true

    - name: Set timezone
      community.general.timezone:
        name: UTC
```

### 🛠️ Creating custom collections

```bash
ansible-galaxy collection init acme.platform
```

```text
ansible_collections/
└── acme/
    └── platform/
        ├── galaxy.yml
        ├── plugins/
        │   ├── modules/
        │   │   └── hello_platform.py
        │   └── lookup/
        │       └── env_prefix.py
        └── roles/
            └── baseline/
```

```yaml
# galaxy.yml
namespace: acme
name: platform
version: 1.0.0
readme: README.md
authors:
  - Platform Team
license:
  - MIT
```

```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule


def main():
    module = AnsibleModule(
        argument_spec={
            'name': {'type': 'str', 'required': True},
        }
    )
    module.exit_json(changed=False, message=f"hello {module.params['name']}")


if __name__ == '__main__':
    main()
```

Custom collections are the best long-term place for internal modules and plugins because they give you namespacing, packaging, and reusable documentation.

### 🧩 Custom modules

Custom modules are appropriate when shell commands become difficult to validate, idempotency is hard to maintain, or an internal API needs a first-class automation interface.

```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule
import json
import os


def read_state(path):
    if not os.path.exists(path):
        return {}
    with open(path, 'r', encoding='utf-8') as handle:
        return json.load(handle)


def write_state(path, data):
    with open(path, 'w', encoding='utf-8') as handle:
        json.dump(data, handle, indent=2)


def main():
    module = AnsibleModule(
        argument_spec={
            'path': {'type': 'path', 'required': True},
            'key': {'type': 'str', 'required': True},
            'value': {'type': 'str', 'required': True},
        },
        supports_check_mode=True,
    )

    path = module.params['path']
    key = module.params['key']
    value = module.params['value']
    state = read_state(path)
    changed = state.get(key) != value

    if module.check_mode:
        module.exit_json(changed=changed, before=state)

    if changed:
        state[key] = value
        write_state(path, state)

    module.exit_json(changed=changed, path=path, key=key, value=value)


if __name__ == '__main__':
    main()
```

### 🔌 Plugins: callback, connection, and lookup

#### Callback plugin example

```python
from ansible.plugins.callback import CallbackBase


class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'result_summary'

    def v2_runner_on_ok(self, result):
        host = result._host.get_name()
        task = result.task_name or result._task.get_name()
        changed = result._result.get('changed', False)
        self._display.display(f"OK host={host} task={task} changed={changed}")

    def v2_runner_on_failed(self, result, ignore_errors=False):
        host = result._host.get_name()
        task = result.task_name or result._task.get_name()
        self._display.display(f"FAILED host={host} task={task} ignore={ignore_errors}")
```

#### Lookup plugin example

```python
from ansible.errors import AnsibleError
from ansible.plugins.lookup import LookupBase
import os


class LookupModule(LookupBase):
    def run(self, terms, variables=None, **kwargs):
        prefix = kwargs.get('prefix', '')
        results = []
        for term in terms:
            value = os.getenv(f"{prefix}{term}")
            if value is None:
                raise AnsibleError(f"Environment variable not found for {prefix}{term}")
            results.append(value)
        return results
```

Connection plugins alter how Ansible reaches targets, callback plugins alter reporting, and lookup plugins pull data into playbooks at runtime.

### 📥 Ansible Pull

`ansible-pull` reverses the normal push model.

Instead of the controller connecting outward, each managed node pulls playbooks from Git and applies them locally.

```bash
ansible-pull -U https://github.com/example/platform-automation.git -C main local.yml
```

This is useful for environments behind NAT, disconnected estates, or host-initiated convergence models.

### Appendix E: Module mini-catalog

#### `stat`

- Primary use: Inspect file existence, ownership, permissions, and checksums.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `slurp`

- Primary use: Read remote file content as base64 when direct access is needed.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `fetch`

- Primary use: Copy files from remote hosts back to the control node.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `archive`

- Primary use: Bundle remote files before transfer or cleanup.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `unarchive`

- Primary use: Extract archives and optionally download them first.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `assemble`

- Primary use: Build a file from multiple fragments.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `tempfile`

- Primary use: Create temporary files or directories on the target when a workflow requires it.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `hostname`

- Primary use: Manage the system hostname cleanly.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `sysctl`

- Primary use: Manage kernel parameters persistently.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `sefcontext`

- Primary use: Define SELinux file context mappings.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `seboolean`

- Primary use: Toggle SELinux booleans safely.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `package_facts`

- Primary use: Gather installed package data.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `service_facts`

- Primary use: Collect service state information.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `uri`

- Primary use: Call REST endpoints for health checks or integrations.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `git`

- Primary use: Clone or update repositories directly on target hosts.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `synchronize`

- Primary use: Use rsync-style transfers when large file trees are involved.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `postgresql_*`

- Primary use: Manage PostgreSQL users, databases, and privileges through purpose-built modules.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `mysql_*`

- Primary use: Manage MySQL and MariaDB resources declaratively.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `k8s`

- Primary use: Interact with Kubernetes resources from Ansible.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.

#### `docker_container`

- Primary use: Manage containers directly where Docker remains the runtime.

- Rule of thumb: if a module exists for the desired system object, prefer it over custom shell logic.

- Review mode support: confirm whether the module supports check mode and diff mode before relying on dry-run results.
