# netactuate-ansible-compute — AI Provisioning Context

Provisions Ubuntu 24.04 LTS VMs on NetActuate's global edge. No BGP, no services — just
compute with SSH access. Give me an API key, locations, count, plan, and SSH key and I'll
have VMs running.

## Required Inputs

| Input | Source | Example |
|-------|--------|---------|
| API key | portal.netactuate.com/account/api | `"abc123..."` |
| Contract ID | Portal API page | `"12345"` |
| SSH public key | Customer's machine | `~/.ssh/id_ed25519.pub` |
| Locations | Customer choice | `LAX, FRA, SIN` |
| Plan | Customer choice | `"VR1x1x25"` |
| Hostname pattern | Customer choice | `"myapp-POP-vm1.example.com"` |

## Files to Edit

### 1. `keys.pub`
Paste the SSH public key content.

### 2. `group_vars/all`
Copy from `group_vars/all.example`:
```yaml
auth_token: "<API_KEY>"
contract_id: "<CONTRACT_ID>"
```
Do not change `operating_system` or `ssh_public_key`.

### 3. `hosts`
```ini
myapp-LAX-vm1.example.com location=LAX plan='VR1x1x25'
myapp-FRA-vm1.example.com location=FRA plan='VR1x1x25'
```

## Commands

```bash
# Setup (one time)
python3 -m venv .venv && source .venv/bin/activate
pip install --upgrade pip && pip install ansible
pip install git+https://github.com/netactuate/naapi.git@vapi2
ansible-galaxy collection install git+https://github.com/netactuate/ansible-collection-compute.git,vapi2

# Provision
ansible-playbook createnode.yaml

# Teardown
ansible-playbook deletenode.yaml
```

## Validation

After `createnode.yaml` completes, SSH to each node:
```bash
ssh ubuntu@<ip-from-host_vars>
```
The IP is in `host_vars/<hostname>` under `ansible_ssh_host`.

## Common Errors

| Error | Fix |
|-------|-----|
| SSH timeout during provision | Re-run `createnode.yaml` — idempotent |
| Collection not found | Run `ansible-galaxy collection install ...` |
| naapi import error | Run `pip install git+...naapi.git@vapi2` in venv |
