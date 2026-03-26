# netactuate-ansible-compute

The fastest path to N virtual machines across M global locations. This Ansible playbook
provisions Ubuntu 24.04 LTS compute nodes on NetActuate's edge network with SSH access
and base packages — ready for any workload in a single command.

## What This Deploys

For each node in your inventory:

1. A NetActuate compute VM running Ubuntu 24.04 LTS
2. SSH key access for both `ubuntu` and `root` users
3. Base packages: net-tools, sysstat, atop
4. Fully updated system (apt upgrade on first boot)

## AI-Assisted (Claude Code / Cursor / Copilot)

Give your AI assistant this prompt — replace placeholders with your values:

```
Provision NetActuate compute VMs for me:

- API Key: <YOUR_API_KEY>
- Contract ID: <YOUR_CONTRACT_ID>
- Locations: LAX, FRA, SIN
- Plan: VR1x1x25
- Hostname pattern: myapp-POP-vm1.example.com

Please:
1. Set up the venv and install dependencies
2. Configure group_vars/all and keys.pub with my SSH key
3. Build the hosts inventory
4. Run createnode.yaml
5. Show me the IPs to SSH into
```

## Prerequisites

### NetActuate Account Requirements

| Requirement | Where to Get It |
|-------------|----------------|
| API key | [portal.netactuate.com/account/api](https://portal.netactuate.com/account/api) — whitelist your IP |
| Contract ID | Visible on the portal API page |

### Control Node Setup

#### macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install ansible
pip install git+https://github.com/netactuate/naapi.git@vapi2
ansible-galaxy collection install git+https://github.com/netactuate/ansible-collection-compute.git,vapi2
```

#### Linux

```bash
sudo apt install python3-venv python3-pip
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install ansible
pip install git+https://github.com/netactuate/naapi.git@vapi2
ansible-galaxy collection install git+https://github.com/netactuate/ansible-collection-compute.git,vapi2
```

#### Windows (WSL2)

Install WSL2 with Ubuntu 24.04, then follow the Linux instructions above.

## Configuration

### Step 1: Add your SSH public key

```bash
cat ~/.ssh/id_ed25519.pub > keys.pub
```

### Step 2: Edit group_vars/all

```bash
cp group_vars/all.example group_vars/all
```

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `auth_token` | string | NetActuate API key | `"abc123def456"` |
| `ssh_public_key` | string | Path to public key file | `"keys.pub"` |
| `operating_system` | string | OS image name | `"Ubuntu 24.04 LTS (20240423)"` |
| `contract_id` | string | Billing contract ID | `"67890"` |

### Step 3: Edit the hosts inventory

One line per VM under `[nodes]`:

```ini
[master]
localhost ansible_connection=local ansible_python_interpreter=.venv/bin/python3

[nodes]
myapp-LAX-vm1.example.com location=LAX plan='VR1x1x25'
myapp-FRA-vm1.example.com location=FRA plan='VR1x1x25'
myapp-SIN-vm1.example.com location=SIN plan='VR1x1x25'
```

## Deployment

```bash
source .venv/bin/activate
ansible-playbook createnode.yaml
```

Re-run if any nodes time out — it is idempotent.

After provisioning, SSH to any node:

```bash
ssh ubuntu@<node-ip>
```

Node IPs are saved in `host_vars/<hostname>` after provisioning.

## Teardown

```bash
ansible-playbook deletenode.yaml
```

Terminates all VMs and cancels billing.

## PoP Location Codes

See [portal.netactuate.com/account/api](https://portal.netactuate.com/account/api) or
[netactuate.com/edge-locations](https://www.netactuate.com/edge-locations/).

## VM Plans

See [portal.netactuate.com/account/api](https://portal.netactuate.com/account/api).

## Related Resources

- [NetActuate API Documentation](https://www.netactuate.com/docs/)
- [NetActuate Ansible Collection](https://github.com/netactuate/ansible-collection-compute)
- [NetActuate Portal](https://portal.netactuate.com)

## Need Help?

- NetActuate support: support@netactuate.com
- API access and billing: [portal.netactuate.com](https://portal.netactuate.com)
