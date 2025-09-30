# RHEL SSH Connectivity Quickstart (Ansible Navigator)

Get started quickly with SSH-based RHEL connectivity using Ansible Navigator and the Ansible Automation Platform 2.5 execution environment.

## What’s included
- `ansible-navigator.yml` – uses `registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel9:latest` and mounts an SSH directory into the execution environment (update the mount path for your user if needed)
- `ansible.cfg` – points to `./inventory` and `~/.ssh/chrislab-aws.pem` as the private key
- `inventory` – example `rhel_aws` group with a host entry
- `group_vars/rhel_aws.yml` – sets `ansible_user: ec2-user` for Amazon Linux/RHEL on AWS
- `test_connection.yml` – simple playbook to verify SSH connectivity and identity
 - `deploy_nginx.yml` – installs NGINX and serves a simple demo page

## Prerequisites (RHEL)
- RHEL host with a container engine (Podman recommended; Docker also works)
- Python tooling to install `ansible-navigator` (pipx or pip)
- Access to pull the EE image from Red Hat registry (or switch to a public image)

```bash
sudo dnf install -y podman python3-pip
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install ansible-navigator   # or: python3 -m pip install --user ansible-navigator

# Optional: authenticate for the default EE image
podman login registry.redhat.io  # or: docker login registry.redhat.io

# Ensure your SSH key permissions are correct
chmod 600 ~/.ssh/chrislab-aws.pem
```

## Configure your RHEL hosts
- Edit `inventory` and place your hosts under the `rhel_aws` group
- Adjust `group_vars/rhel_aws.yml` if your remote user differs from `ec2-user`
- Confirm your key exists at `~/.ssh/chrislab-aws.pem` (as referenced by `ansible.cfg`)

## Test connectivity
Run the included test playbook using Ansible Navigator (uses the EE container and your mounted `~/.ssh` automatically):

```bash
ansible-navigator run test_connection.yml -m stdout
```

You should see a successful `ping` and a message reporting the connected user.

## Deploy the demo NGINX page
Run the demo playbook to install NGINX and deploy a simple index page:

```bash
ansible-navigator run deploy_nginx.yml -m stdout
```

Customize the message shown on the page with an extra var:

```bash
ansible-navigator run deploy_nginx.yml -e demo_message="Hello from Ansible" -m stdout
```

## Troubleshooting (quick)
- SSH key issues: confirm permissions `chmod 600 ~/.ssh/chrislab-aws.pem`
- Network: ensure security groups/firewalls allow TCP 22 to your hosts
- Username: set `ansible_user` in `group_vars/rhel_aws.yml` if not `ec2-user`
- Registry access: if `registry.redhat.io` is unavailable, change the image in `ansible-navigator.yml` to `quay.io/ansible/creator-ee:latest`
- Podman on RHEL: verify Podman is installed and working (e.g., `podman info`)