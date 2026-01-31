# Azure DevOps Build Agent - Ansible

Ansible playbook and Docker setup for provisioning an Azure DevOps self-hosted build agent.

## Two Deployment Methods

### 1. Ansible Playbook (VM)

Installs the Azure DevOps agent directly on a Linux VM as a systemd service.

**What it does:**
- Installs required packages (`curl`, `jq`, `libicu-dev`, `libssl-dev`, etc.)
- Downloads Azure Pipelines agent v4.258.1
- Configures agent against Azure DevOps org with PAT authentication
- Registers and starts as a systemd service
- Installs Docker on the target VM

**Usage:**

```bash
export AZP_TOKEN=<your-pat-token>
cd azure-agent-ansible
ansible-playbook playbook.yml --ask-become-pass --extra-vars "azp_token=$AZP_TOKEN"
```

### 2. Docker Container

Runs the agent as a containerized process, self-configuring on startup.

**Build and run:**

```bash
docker build -t azure-devops-agent -f azure-agent-ansible/files/Dockerfile .
docker run -d \
  -e AZP_URL="https://dev.azure.com/<your-org>" \
  -e AZP_TOKEN="<your-pat>" \
  -e AZP_POOL="Default" \
  azure-devops-agent
```

## Project Structure

```
azure-agent-ansible/
  ansible.cfg           # Ansible configuration
  inventory.ini         # Target hosts
  playbook.yml          # Main agent deployment playbook
  test_playbook.yml     # SSH connectivity test
  files/
    Dockerfile          # Containerized agent (Ubuntu 20.04)
    start.sh            # Container entrypoint script
```

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `AZP_URL` | `https://dev.azure.com/kamilkural` | Azure DevOps organization URL |
| `AZP_TOKEN` | env var | Personal Access Token |
| `AZP_POOL` | `Default` | Agent pool name |
| `azp_agent_version` | `4.258.1` | Agent version |

## Prerequisites

- Ansible 2.9+
- SSH access to target VM
- Azure DevOps PAT with Agent Pools (Read & manage) scope
