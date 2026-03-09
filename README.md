# K3s with Cilium - Ansible Deployment

Ansible playbooks to deploy a K3s Kubernetes cluster with Cilium CNI on Linux VMs.

## Features

- Lightweight Kubernetes (K3s) installation
- Cilium CNI with eBPF dataplane
- Hubble observability (UI and Relay)
- kube-proxy replacement via Cilium
- Multi-node cluster support

## Requirements

### Control Machine
- Ansible 2.12+
- Python 3.8+

### Target VMs
- Ubuntu 22.04 / Debian 12 (recommended)
- 2+ vCPUs, 2GB+ RAM per node
- SSH key-based authentication
- Sudo privileges

## Quick Start

### 1. Install Ansible dependencies

```bash
pip install ansible
ansible-galaxy collection install -r requirements.yml
```

### 2. Configure inventory

Edit `inventory/hosts.ini` with your VM details:

```ini
[k3s_server]
server1 ansible_host=192.168.1.10

[k3s_agents]
agent1 ansible_host=192.168.1.11
agent2 ansible_host=192.168.1.12

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 3. Customize variables (optional)

Edit `group_vars/all.yml` to adjust versions and settings.

### 4. Run the playbook

```bash
# Test connectivity first
ansible all -m ping

# Deploy the cluster
ansible-playbook site.yml
```

### 5. Access the cluster

After deployment, the kubeconfig is saved to `./kubeconfig`:

```bash
export KUBECONFIG=$(pwd)/kubeconfig
kubectl get nodes
kubectl -n kube-system get pods
```

## Playbooks

| Playbook | Description |
|----------|-------------|
| `site.yml` | Full cluster deployment |
| `reset.yml` | Completely remove K3s from all nodes |

## Roles

| Role | Description |
|------|-------------|
| `prereqs` | System preparation (packages, kernel modules, sysctl) |
| `k3s_server` | K3s control plane installation |
| `k3s_agent` | K3s worker node installation |
| `cilium` | Cilium CNI installation via Helm |

## Verification

After deployment, verify Cilium status:

```bash
# Check Cilium pods
kubectl -n kube-system get pods -l app.kubernetes.io/part-of=cilium

# Install Cilium CLI (optional)
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-amd64.tar.gz
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin

# Run connectivity test
cilium status
cilium connectivity test
```

## Accessing Hubble UI

Port-forward to access the Hubble UI:

```bash
kubectl -n kube-system port-forward svc/hubble-ui 12000:80
# Open http://localhost:12000 in your browser
```

## Customization

### K3s Version

Edit `group_vars/all.yml`:

```yaml
k3s_version: "v1.29.0+k3s1"
```

### Cilium Version

Edit `group_vars/all.yml`:

```yaml
cilium_version: "1.15.0"
```

### Network CIDRs

Edit `group_vars/all.yml`:

```yaml
cluster_cidr: "10.42.0.0/16"
service_cidr: "10.43.0.0/16"
```

## Troubleshooting

### Check K3s logs

```bash
# On server node
sudo journalctl -u k3s -f

# On agent nodes
sudo journalctl -u k3s-agent -f
```

### Check Cilium logs

```bash
kubectl -n kube-system logs -l app.kubernetes.io/name=cilium-agent -f
```

### Reset and reinstall

```bash
ansible-playbook reset.yml
ansible-playbook site.yml
```

## License

MIT
