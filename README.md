# Durcissement (CIS Hardening) Ansible Project

This repository contains Ansible roles and playbooks to apply CIS-style hardening, run audits and generate compliance reports (JSON and HTML). It is designed to work on Debian/Ubuntu systems and be tolerant when executed in container environments (Docker).

## Repository Structure

- `playbooks/`
  - `cis_hardening.yml` — main playbook to apply hardening rules (role `hardening`).
  - `audit.yml` — playbook that runs the `audit` role and collects reports.
  - other helper playbooks (can be removed if unused).
- `roles/hardening/` — hardening role (many CIS rules consolidated into one `main.yml`).
- `roles/audit/` — audit role that checks system state, generates JSON/HTML reports and fetches them to the controller node.
- `roles/*/templates/` — Jinja2 templates used to generate reports (JSON and HTML templates).
- `inventory/` — sample inventory (`hosts`).
- `ansible.cfg` — recommended Ansible configuration for running this project.
- `logs/` — Ansible run logs (ignored in .gitignore).

## Quick Start

Prerequisites:
- Ansible 2.9+ (recent versions recommended).
- SSH access to target hosts.

Run hardening only:
```powershell
ansible-playbook playbooks/cis_hardening.yml -i inventory/hosts
```

Run hardening and then generate & collect audit reports (JSON + HTML):
```powershell
ansible-playbook playbooks/cis_hardening.yml -i inventory/hosts -e "run_audit_after_hardening=true"
```

Run audit first (baseline) then hardening (default can be configured in the playbook variables):
```powershell
ansible-playbook playbooks/cis_hardening.yml -i inventory/hosts -e "run_pre_audit=true"
```

Or run the dedicated audit playbook (separate):
```powershell
ansible-playbook playbooks/audit.yml -i inventory/hosts
```

## Reports

- By default reports are fetched to the controller (the machine you run Ansible from) under:
  - `{{ playbook_dir }}/reports/<inventory_hostname>/audit_report.json`
  - `{{ playbook_dir }}/reports/<inventory_hostname>/audit_report.html`

- If you run Ansible from inside a container, mount the project directory as a volume so the `reports/` directory is persisted on the host.

## Important Variables

- `run_pre_audit` (bool) — run audit before applying hardening (default configured in `cis_hardening.yml`).
- `run_audit_after_hardening` (bool) — run audit after hardening and collect reports.
- `audit_reports_local_dir` — local path where fetched reports are saved (default `{{ playbook_dir }}/reports`).
- `skip_firewall`, `skip_aide`, `skip_sudo_hardening` — control optional sections in the `hardening` role.

All variables can be overridden with `-e` on the `ansible-playbook` command line or through `group_vars` / `host_vars`.

## Docker / CI Notes

- Several tasks are guarded to be safe in container environments (they check `ansible_facts['virtualization_type'] == 'docker'`).
- `ansible.cfg` in the repository enables `pipelining` and increased `forks` for performance — if sudo prompts are expected, disable pipelining.
- For persistence of reports or logs when running in Docker, mount a host directory into the container at the project root.

## Handlers and Notifications

- Handlers such as `Restart auditd`, `Restart SSH`, `Restart UFW` are defined in role handlers. Do not remove these handlers unless you update corresponding `notify:` calls across roles.

## Linting & Syntax Checks

- Before running in production, consider running `ansible-lint` and `ansible-playbook --syntax-check` on the playbooks and roles.

## Contributing

- Add tests or sample inventories under `inventory/`
---

