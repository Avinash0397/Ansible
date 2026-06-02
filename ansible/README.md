# RKE2 HA Kubernetes on Azure with Ansible

Production-ready Ansible project to deploy a 3-node Highly Available RKE2 Kubernetes cluster on Azure using embedded etcd.

## Architecture

- **Cloud**: Azure
- **OS**: Ubuntu 24.04 LTS
- **Topology**: 3 VMs across Availability Zones (AZ1/AZ2/AZ3)
- **Role of each VM**: `rke2-server` (control-plane + etcd on all nodes)
- **Datastore**: Embedded etcd (quorum-based HA)
- **Kubernetes API**: `6443/tcp`
- **RKE2 registration**: `9345/tcp`
- **Failure tolerance**: survives any single node failure (2/3 etcd quorum remains)

## Project Tree

```text
ansible/
├── ansible.cfg
├── requirements.yml
├── inventory/
│   ├── dev.ini
│   └── prod.ini
├── group_vars/
│   └── all.yml
├── host_vars/
│   ├── vm1.yml
│   ├── vm2.yml
│   └── vm3.yml
├── playbooks/
│   └── site.yml
├── roles/
│   ├── common/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   ├── rke2_primary/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   ├── rke2_join/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   ├── kubectl/
│   │   └── tasks/
│   │       └── main.yml
│   └── validation/
│       └── tasks/
│           └── main.yml
├── templates/
│   ├── rke2-join-config.yaml.j2
│   └── rke2-primary-config.yaml.j2
└── files/
    └── .gitkeep
```

## Prerequisites

1. Azure VMs provisioned:
   - `vm1` in Availability Zone 1
   - `vm2` in Availability Zone 2
   - `vm3` in Availability Zone 3
2. Private IP connectivity between all nodes.
3. SSH access from Ansible control node to all VMs.
4. Python 3 on all target hosts.
5. Required ports open in NSG and host firewall:
   - `22/tcp`, `6443/tcp`, `9345/tcp`, `2379-2380/tcp`, `10250/tcp`, `8472/udp`

## Inventory Examples

Edit `inventory/prod.ini` with your real private IPs:

```ini
[rke2_primary]
vm1 ansible_host=10.20.1.4

[rke2_join]
vm2 ansible_host=10.20.2.4
vm3 ansible_host=10.20.3.4

[rke2_servers:children]
rke2_primary
rke2_join

[all:vars]
ansible_user=azureadmin
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_become=true
ansible_python_interpreter=/usr/bin/python3
```

## Variables Documentation

Primary variable file: `group_vars/all.yml`

- `cluster_name`: logical cluster name.
- `timezone`: node timezone.
- `rke2_api_server_port`: Kubernetes API port (default `6443`).
- `rke2_registration_port`: RKE2 server registration port (default `9345`).
- `rke2_cluster_cidr`: pod CIDR.
- `rke2_service_cidr`: service CIDR.
- `rke2_tls_sans`: SANs added to API cert.
- `rke2_etcd_snapshot_schedule_cron`: automatic etcd snapshot cron schedule.
- `rke2_etcd_snapshot_retention`: number of retained snapshots.
- `rke2_wait_retries` / `rke2_wait_delay`: retry controls for readiness checks.
- `required_packages`: base OS packages.
- `required_sysctl`: kernel networking settings.
- `cluster_admin_user`: user receiving kubeconfig on primary node.
- `kubeconfig_server_endpoint`: endpoint written into admin kubeconfig.
- `validation_required_nodes`: expected node count.
- `azure_*`: vault-protected placeholders for secrets.

Host-specific variables in `host_vars/vm*.yml`:

- `node_name`
- `rke2_node_ip`
- `rke2_advertise_address`
- `availability_zone`

## Role Responsibilities

### `common`

- OS update and package install
- swap disable (runtime and persistent)
- kernel modules and sysctl configuration
- time sync (chrony) and timezone
- UFW firewall rules for RKE2 and Kubernetes

### `rke2_primary`

- install RKE2 server on first node
- render server config
- start/enable `rke2-server`
- wait for ports and bootstrap completion
- read `/var/lib/rancher/rke2/server/node-token`
- store token as cached Ansible fact

### `rke2_join`

- fetch bootstrap token from primary host facts
- render join config on VM2/VM3
- install RKE2 server
- start/enable `rke2-server`
- validate node readiness

### `kubectl`

- export RKE2 kubectl path in `/etc/profile.d`
- copy kubeconfig for cluster admin user
- patch kubeconfig endpoint for remote access
- validate `kubectl get nodes`

### `validation`

- assert expected node count
- assert all nodes report `Ready=True`
- verify etcd health
- verify control-plane pod health
- verify CoreDNS, metrics-server, and ingress controller
- print final cluster status

## Playbook Execution Guide

Run from `ansible/`:

1. Install collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

2. Encrypt secrets (replace placeholders first):

```bash
ansible-vault encrypt group_vars/all.yml
```

3. Validate inventory reachability:

```bash
ansible -i inventory/prod.ini rke2_servers -m ping
```

4. Deploy cluster:

```bash
ansible-playbook -i inventory/prod.ini playbooks/site.yml --diff
```

5. Safe re-run for idempotency verification:

```bash
ansible-playbook -i inventory/prod.ini playbooks/site.yml --diff
```

## Validation Guide

After successful playbook execution:

```bash
ssh azureadmin@<vm1_private_ip>
sudo /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get nodes
```

Expected result:

```text
vm1   Ready   control-plane,etcd   ...
vm2   Ready   control-plane,etcd   ...
vm3   Ready   control-plane,etcd   ...
```

Additional checks:

```bash
sudo /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml -n kube-system get pods
sudo /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml -n kube-system get endpoints
```

## Troubleshooting Guide

1. **Node does not join**
   - check token on primary: `sudo cat /var/lib/rancher/rke2/server/node-token`
   - check join node config: `/etc/rancher/rke2/config.yaml`
   - verify `9345/tcp` reachability between nodes

2. **API not reachable on 6443**
   - check primary service: `sudo systemctl status rke2-server`
   - inspect logs: `sudo journalctl -u rke2-server -f`
   - confirm firewall/NSG rules allow `6443/tcp`

3. **etcd unhealthy**
   - verify at least 2 nodes are up
   - inspect etcd pod logs in `kube-system`
   - check node clock sync (`chrony`) and network latency

4. **Pods stuck Pending/CrashLoopBackOff**
   - inspect node resources: `kubectl top nodes` (if metrics available)
   - inspect events: `kubectl get events -A --sort-by=.lastTimestamp`

## Failure Handling and Rollback Guidance

- If Stage 3/4 fails for `vm2` or `vm3`, fix issue and rerun playbook; `serial: 1` ensures controlled incremental join.
- If a node is partially configured, rerun `site.yml`; tasks are idempotent and handlers restart services only on config change.
- If a join node must be rebuilt:
  1. Replace VM in Azure.
  2. Update inventory/host_vars for new IP.
  3. Re-run `site.yml` to rejoin node.
- If primary node fails permanently, restore using etcd snapshot (see recovery section).

## HA Testing Procedure

1. Confirm steady state:
   - all 3 nodes `Ready`.
2. Simulate single-node failure:
   - stop one node (or stop `rke2-server`) on `vm3`.
3. Validate quorum and API:
   - run `kubectl get nodes` from surviving node.
   - confirm workloads remain available.
4. Recover failed node:
   - start node/service again.
   - verify node returns to `Ready`.
5. Repeat for a different node to validate AZ resilience.

## Cluster Recovery Guide

### A. Restore from automatic etcd snapshot

1. Identify snapshot on a surviving control-plane node:

```bash
sudo ls -lah /var/lib/rancher/rke2/server/db/snapshots
```

2. Stop RKE2 service on all nodes:

```bash
sudo systemctl stop rke2-server
```

3. Restore snapshot on the node chosen for recovery:

```bash
sudo rke2 server \
  --cluster-reset \
  --cluster-reset-restore-path=/var/lib/rancher/rke2/server/db/snapshots/<snapshot-name>
```

4. Start RKE2 on recovery node, then on other nodes:

```bash
sudo systemctl start rke2-server
```

5. Validate cluster health with `kubectl get nodes` and `kubectl -n kube-system get pods`.

### B. Lost one node

- Recreate VM in same or different AZ.
- Reapply base networking + host_vars.
- Run playbook; node rejoins using token from primary.

## Security Notes

- Replace all vault placeholders in `group_vars/all.yml` before production.
- Restrict SSH source ranges in NSG.
- Prefer private load balancer/DNS VIP for API endpoint SAN and kubeconfig endpoint.
- Rotate RKE2 token and certificates according to platform policy.

## Operational Notes

- This repository deploys an RKE2-native cluster only (no Kind, no external bootstrap cluster).
- All nodes are server/control-plane nodes with embedded etcd.
- Playbooks are designed for safe re-runs and production operations.
