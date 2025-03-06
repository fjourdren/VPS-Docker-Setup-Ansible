# Ansible Security Playbook

This playbook implements security hardening measures using the geerlingguy.security role along with custom configurations for user management, docker + docker-compose install using geerlingguy.docker and run a nginx server through a docker-compose.

## Prerequisites

- Ansible 2.9 or higher
- Root access to target servers
- SSH access configured
- WSL with Ubuntu distribution (if running from Windows)

## Installation

1. Install required roles:

```bash
ansible-galaxy install -r requirements.yml
```

2. Configure your inventory file with target hosts under the `provisionning` group.

## Usage

### Provision

If using Windows with WSL:

```bash
wsl --distribution Ubuntu
ansible-playbook provision.yml -i hosts
```

Use it using a ssh key :

```bash
ansible-playbook provision.yml -i hosts --key-file ~/.ssh/id_rsa
```

Remember to update the hosts file with the new SSH port and user for deploy playbook runs (just copy the docker-compose group host into the provisionning one).

### Deploy

```bash
wsl --distribution Ubuntu
ansible-playbook deploy.yml -i hosts --key-file ~/.ssh/id_rsa
```

## Configuration

### Main Security Features

- SSH Hardening (Custom port: 30000)
- Automatic security updates with reboot scheduling
- UFW Firewall configuration
- fail2ban implementation
- Email notifications for security updates

### Variables

Key variables are centralized for easier management:

```yaml
user:
  username: ansible
  password: [hashed-password]
  public_key: ~/.ssh/id_rsa.pub

ssh:
  port: 30000
```

To crypt your password :

```bash
ansible all -i localhost, -m debug -a "msg={{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"
```

### Firewall Rules

Default allowed ports:

- SSH (30000)
- HTTP (80)
- HTTPS (443)

### Automatic Updates

- Enabled with automatic reboot
- Reboot time: 06:00
- Email notifications configured for errors

### fail2ban Configuration

- Ban time: 600 seconds
- Find time: 600 seconds
- Max retry: 3 attempts

## Roles

### Provisionning

Used during the initial Ansible run, primarily to update the SSH port.

- `common`: Base system configuration
- `user`: User management
- `geerlingguy.security`: Security hardening
- `geerlingguy.docker`: Install docker

### Docker-compose

To apply changes every time we modify the docker-compose file.

- `run-docker-compose`: Web server configuration with a docker-compose
