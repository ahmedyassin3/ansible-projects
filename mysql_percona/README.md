# MySQL Percona Ansible Playbooks

This folder contains Ansible playbooks and roles to install Percona Server for MySQL on RedHat-based systems.

Structure
- `playbook_57.yml` - Installs Percona Server 5.7 using the `mysql57` role.
- `playbook_80.yml` - Installs Percona Server 8.x using the `mysql80` role. Prompts for `percona_version`.
- `playbook_apply_patches.yml` - Applies system updates via `yum` (uses `state: latest`).
- `roles/common/tasks/main.yml` - Prepares the host: installs Python3, `pip`, PyMySQL, and configures the Percona yum repository.
- `roles/mysql57/tasks/main.yml` - Installs Percona Server 5.7 and starts/enables the `mysqld` service.
- `roles/mysql80/tasks/main.yml` - Enables the Percona 8.0 repo and installs the requested `percona_version`, then starts/enables `mysqld`.

Prerequisites
- Ansible installed on the control host.
- Target hosts are RedHat-compatible (yum/dnf-based).
- Playbooks assume `become: yes` to perform package and service operations.

Usage

Install Percona 5.7 (runs `common` then `mysql57`):

```bash
ansible-playbook playbook_57.yml -i inventory.ini
```

Install Percona 8.x (prompts for version):

```bash
ansible-playbook playbook_80.yml -i inventory.ini
# When prompted, enter a value like: 8.0.32
```

Apply OS patches on hosts:

```bash
ansible-playbook playbook_apply_patches.yml -i inventory.ini
```

Notes
- The `mysql80` role expects the variable `percona_version` to match a valid Percona package suffix (e.g., `8.0.32`).
- The `common` role configures the Percona yum repository by adding a repository entry that points to the Percona RPM installer; verify network access to `repo.percona.com`.