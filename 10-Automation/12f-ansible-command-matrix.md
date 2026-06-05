# Ansible Command Matrix

← Back to [12-ansible-deep-dive.md](./12-ansible-deep-dive.md)

The original appendix repeated the same ten command patterns one hundred times. This file keeps the unique patterns once so the reference stays useful without boilerplate duplication.

---

## Core command patterns

#### Preview hosts
```bash
ansible-playbook site.yml --list-hosts
```

#### Preview tasks
```bash
ansible-playbook site.yml --list-tasks
```

#### Check mode run
```bash
ansible-playbook site.yml --check
```

#### Diff mode run
```bash
ansible-playbook site.yml --diff
```

#### Tag execution
```bash
ansible-playbook site.yml --tags web
```

#### Skip tags
```bash
ansible-playbook site.yml --skip-tags reboot
```

#### Single host limit
```bash
ansible-playbook site.yml --limit web1.example.com
```

#### Serial rollout
```bash
ansible-playbook site.yml --limit web --serial 1
```

#### Fact query
```bash
ansible all -m setup -a 'filter=ansible_default_ipv4'
```

#### Inventory graph
```bash
ansible-inventory -i inventories/prod/hosts.yml --graph
```
