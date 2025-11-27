# Kubespray Hetzner Cluster

This repository automates provisioning bare-metal Hetzner servers and installing a Kubernetes cluster using Kubespray.

## 1. Install Ansible Galaxy Requirements

```
ansible-galaxy install -r requirements.yaml
```

## 2. Configure Secrets

Create `secrets.yaml` and store your Hetzner Cloud API token. This file is consumed by the provisioning playbook.

```yaml
hcloud_token: "<your_hcloud_api_token>"
```

> Tip: keep this file out of version control (e.g., list it in `.gitignore`).

## 3. Provision Servers on Hetzner

Run the provisioning playbook to create or configure the target hosts (SSH keys, networking, etc.).

```
ansible-playbook provision-playbook.yaml
```

## 4. Create the Kubespray Inventory

Update `inventory.yaml` with the public IPs and SSH users of the Hetzner nodes. The file should follow the template below; replace the example IPs with the ones returned by Hetzner and keep `node1` as the first control-plane node.

```yaml
servers:
  hosts:
    node1:
      ansible_host: <node 1 ip>
      ansible_user: root
    node2:
      ansible_host: <node 2 ip>
      ansible_user: root

kube_control_plane:
  children:
    servers:

etcd:
  hosts:
    node1:

kube_node:
  children:
    servers:
```

## 5. Install Kubernetes with Kubespray

Execute the Kubespray installation playbook using the inventory you prepared. This step bootstraps Kubernetes across the Hetzner nodes.

```
ansible-playbook -i inventory.yaml install-k8s-playbook.yaml
```

## Next Steps

- Verify the cluster by copying the generated `admin.conf` to your local `~/.kube/config` and running `kubectl get nodes`.
- Adjust `group_vars` or `host_vars` to enable additional Kubespray features (e.g., metrics server, ingress controller) as needed.
