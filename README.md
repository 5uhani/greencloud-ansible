# AGCP — Baseline Automation (Member 5)

This covers your Phase 1 / Review 1 technical task:
**"Create initial Ansible playbooks for instance bootstrapping and configuration management."**

## What's in here

```
agcp-ansible/
├── ansible.cfg              # Ansible settings (points to inventory)
├── site.yml                 # Main playbook — run this
├── inventory/
│   └── hosts.ini            # List of target machines (edit this)
├── group_vars/
│   └── all.yml               # Shared variables (versions, packages)
└── roles/
    ├── common/               # OS updates, base packages, users, timezone
    ├── security/             # Firewall (ufw) + SSH hardening
    ├── node_exporter/        # Installs Prometheus Node Exporter
    └── docker/                # Installs Docker + Compose (app/db servers only)
```

**What each role actually does, in plain English:**
- **common** = the "bootstrapping" part. Updates the OS, installs basic tools (curl, git, pip, etc.), creates a non-root deploy user, sets timezone/clock sync.
- **security** = basic "configuration management." Opens only the ports you need (SSH, HTTP, FastAPI port, Node Exporter port) and turns off risky SSH defaults.
- **node_exporter** = installs the agent that exposes CPU/RAM/disk metrics, so Member 1's Prometheus server has something to scrape on every machine.
- **docker** = installs Docker only on app/db servers, so Member 3's FastAPI app and Member 4's Postgres/Redis can just run as containers later.

You don't have to keep all four roles if your group wants to scope it down — `common` + `security` alone already satisfies "bootstrapping and configuration management." The other two just make the playbook obviously useful to the rest of the team, which is good for your presentation.

## Easiest way to actually run and demo this

You don't need real AWS/Azure servers to prove this works. Do it locally first, then point it at real cloud instances once Member 2's Terraform is ready.

### Option A — Test locally with Vagrant (recommended, free, no cloud account needed)

The `Vagrantfile` and `inventory/hosts.ini` in this folder are already set up to work together — no editing required for your first test run.

1. Install [VirtualBox](https://www.virtualbox.org/) and [Vagrant](https://developer.hashicorp.com/vagrant/downloads).
2. From inside the `agcp-ansible/` folder, run:
   ```bash
   vagrant up
   ```
   Wait a few minutes — this downloads and boots a small Ubuntu virtual machine on your laptop.
3. Install Ansible on your own laptop (not the VM):
   ```bash
   pip install ansible
   # or on Mac: brew install ansible
   ```
4. Still inside `agcp-ansible/`, run:
   ```bash
   ansible-playbook site.yml
   ```
5. Confirm it worked:
   ```bash
   vagrant ssh
   curl localhost:9100/metrics   # should print a wall of metrics
   docker --version              # should print a version number
   exit
   ```

### Option B — Point directly at real cloud instances (once Terraform is up)

1. Get the IPs from Member 2:
   ```bash
   terraform output -json
   ```
2. Paste those IPs into `inventory/hosts.ini` under the right group (`app_servers`, `db_servers`, `monitoring_servers`).
3. Update `ansible_ssh_private_key_file` to your actual `.pem` key.
4. Test connectivity first:
   ```bash
   ansible all_nodes -m ping
   ```
5. Then run the real thing:
   ```bash
   ansible-playbook site.yml
   ```

## Install Ansible itself (control machine only — not the target servers)

```bash
# macOS
brew install ansible

# Ubuntu/Debian (WSL is fine)
sudo apt update && sudo apt install -y ansible

# Or via pip, anywhere
pip install ansible
```

You do NOT install Ansible on the target instances — it connects over SSH and runs everything remotely. That's the whole point of it.

## For the paper section (Member 5 also owns this for Review 1)

Your "Baseline Automation" technical work pairs with a separate deliverable: the **initial Mathematical Optimization Model** (cost vs. carbon vs. latency objective functions) for the paper. That's a different, math-focused task — happy to help you draft that too if you want, it's a separate piece of work from these playbooks.

## Quick talking points for your review

- "Bootstrapping" = common role (updates, packages, users).
- "Configuration management" = security role (firewall/SSH) + node_exporter/docker roles that configure services consistently across every instance, which is the core idea of Ansible (idempotent, repeatable config vs. manual setup).
- This is "initial" on purpose — Review 2 asks you to extend it to instance resizing, remediation, and rollback policies, which builds directly on top of this file structure (you'll just add new roles like `remediation/` and reuse `site.yml`).
