# raspi-ai-bootstrap

Bootstrap a Raspberry Pi 5 into a self-hosted AI node using Ansible. One command takes a fresh Raspberry Pi OS Lite install (SSH only) to a fully configured machine with Docker, Node.js, and Claude Code.

## What gets installed

| Role | What it does |
|------|-------------|
| **base** | System upgrade, essential packages (git, curl, vim, htop, tmux, jq...), locale, timezone, hostname, UFW firewall (SSH only) |
| **docker** | Docker CE from official repo, Compose plugin, user added to docker group |
| **claude-code** | Node.js LTS via NodeSource, `@anthropic-ai/claude-code` globally via npm |

## Prerequisites

- A Raspberry Pi 5 running **Raspberry Pi OS Lite (64-bit)** with SSH enabled
- SSH access from your machine to the Pi (key-based auth recommended)
- Ansible installed on your local machine:
  ```bash
  # macOS
  brew install ansible

  # Ubuntu/Debian
  sudo apt install ansible
  ```
- Ansible collections:
  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```

## Quick start

### 1. Configure your Pi's connection

Edit `inventory/hosts.yml` with your Pi's IP and SSH user:

```yaml
all:
  hosts:
    raspi:
      ansible_host: 192.168.1.100        # Your Pi's IP
      ansible_user: pi                    # Your SSH user
      ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

### 2. Customize settings (optional)

All settings can be overridden directly in `inventory/hosts.yml`. Uncomment and change any value:

```yaml
      # base_hostname: raspi-ai
      # base_locale: en_US.UTF-8
      # base_timezone: Europe/Paris
      # nodejs_major_version: "22"
```

See each role's `defaults/main.yml` for the full list of variables.

### 3. Test the connection

```bash
ansible all -m ping
```

Expected output:
```
raspi | SUCCESS => { "ping": "pong" }
```

### 4. Run the bootstrap

```bash
# Dry-run first (shows what would change, applies nothing)
ansible-playbook playbooks/bootstrap.yml --check

# Run for real
ansible-playbook playbooks/bootstrap.yml
```

### 5. Configure Claude Code API key

The API key is managed securely with [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html). Create an encrypted vars file:

```bash
ansible-vault create inventory/group_vars/all/vault.yml
```

Add your key inside:

```yaml
anthropic_api_key: "sk-ant-..."
```

Then run the playbook with `--ask-vault-pass`:

```bash
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass
```

The key will be deployed to `/etc/profile.d/claude.sh` (mode `0600`, root only). If you skip this step, the file is created with a commented-out placeholder instead.

> **Tip:** To avoid typing the vault password every time, store it in a file and reference it:
> ```bash
> echo 'my-vault-password' > .vault_pass
> ansible-playbook playbooks/bootstrap.yml --vault-password-file .vault_pass
> ```
> `.vault_pass` is already in `.gitignore`.

## Running a single role

```bash
ansible-playbook playbooks/bootstrap.yml --tags base
ansible-playbook playbooks/bootstrap.yml --tags docker
ansible-playbook playbooks/bootstrap.yml --tags claude-code
```

## Testing

The project uses [Molecule](https://ansible.readthedocs.io/projects/molecule/) for integration testing. It runs the full playbook against a Debian Bookworm Docker container and validates:

- **Converge**: all tasks execute without errors
- **Idempotence**: a second run produces zero changes
- **Verify**: all expected packages and tools are installed

```bash
pip install molecule molecule-plugins[docker]
molecule test
```

## License

Apache 2.0
