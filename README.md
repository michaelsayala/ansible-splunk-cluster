# Ansible Splunk Cluster

Ansible automation for deploying and configuring a distributed Splunk Enterprise environment.

This project automates the installation, configuration, clustering, security, and connectivity of multiple Splunk Enterprise components, including an Indexer Cluster, Search Head Cluster, Deployment Server, Search Head Deployer, License Manager, Heavy Forwarders, and Universal Forwarders.

The project is designed to provide a repeatable way to build a complete Splunk environment while keeping component configuration separated into reusable Ansible roles.

---

## Architecture

The environment consists of the following Splunk components:

| Component            | Hosts                  | Purpose                                                            |
| -------------------- | ---------------------- | ------------------------------------------------------------------ |
| Cluster Manager      | `cm1`                  | Manages the Splunk Indexer Cluster                                 |
| Indexers             | `idx1`, `idx2`, `idx3` | Store and index Splunk data                                        |
| Search Head Deployer | `dp1`                  | Manages configuration and applications for the Search Head Cluster |
| Search Heads         | `sh1`, `sh2`, `sh3`    | Provides distributed search and Splunk Web access                  |
| Deployment Server    | `ds1`                  | Manages configuration deployment to deployment clients             |
| License Manager      | `lm1`                  | Centralized Splunk license management                              |
| Heavy Forwarders     | `hf1`, `hf2`           | Receive, process, and forward data                                 |
| Universal Forwarders | `uf1`, `uf2`           | Collect and forward data                                           |

# Repository Structure

```text
ansible-splunk-cluster/
│
├── group_vars/
│   ├── all.yml.example
│   └── universal_forwarders.yml
│
├── inventory.yml.example
│
├── roles/
│   ├── firewall/
│   ├── splunk_all_outputs_idxc_discovery/
│   ├── splunk_cluster_manager/
│   ├── splunk_common/
│   ├── splunk_deployer/
│   ├── splunk_deployment_server/
│   ├── splunk_hosts_file/
│   ├── splunk_indexer_cluster/
│   ├── splunk_indexer_cluster_discovery/
│   ├── splunk_indexer_receiving/
│   ├── splunk_install/
│   ├── splunk_install_universal_forwarder/
│   ├── splunk_license_manager/
│   ├── splunk_license_peer/
│   ├── splunk_search_head_cluster/
│   ├── splunk_search_head_indexer_cluster/
│   ├── splunk_shc_outputs_idxc_discovery/
│   └── splunk_ssl/
│
└── site.yml.example
```

---

## Ansible Roles

### `splunk_common`

Provides common configuration and operating-system preparation shared by Splunk Enterprise components.

This role is intended to establish the common requirements before component-specific configuration is applied.

### `splunk_install`

Installs Splunk Enterprise on the Splunk Enterprise hosts.

The configured version is:

```yaml
splunk_version: "9.4.10"
```

The default installation location is:

```yaml
splunk_home: "/opt/splunk"
```

The Splunk service runs under:

```yaml
splunk_user: "splunk"
splunk_group: "splunk"
```

### `splunk_install_universal_forwarder`

Installs Splunk Universal Forwarder on forwarder hosts.

Universal Forwarders are managed separately from Splunk Enterprise because their installation and deployment requirements differ from full Splunk Enterprise instances.

### `splunk_cluster_manager`

Configures the Splunk Cluster Manager responsible for managing the Indexer Cluster.

### `splunk_indexer_cluster`

Configures the indexer nodes as members of the Indexer Cluster.

The current inventory contains three indexers:

```text
idx1
idx2
idx3
```

### `splunk_indexer_receiving`

Configures Splunk instances that receive forwarded data.

This role is used for both Indexers and Heavy Forwarders.

### `splunk_indexer_cluster_discovery`

Configures Indexer Discovery through the Cluster Manager.

This allows Splunk components to discover available indexers dynamically rather than relying exclusively on hard-coded indexer addresses.

### `splunk_search_head_cluster`

Configures the Search Head Cluster.

The current environment contains:

```text
sh1
sh2
sh3
```

### `splunk_search_head_indexer_cluster`

Connects the Search Head Cluster to the Indexer Cluster.

This separates Search Head Clustering from the configuration required for distributed search against the Indexer Cluster.

### `splunk_deployer`

Configures the Search Head Deployer.

The Deployer is responsible for distributing applications and configuration to Search Head Cluster members.

### `splunk_deployment_server`

Configures the Splunk Deployment Server.

The Deployment Server manages configuration deployment to clients such as Universal Forwarders.

### `splunk_license_manager`

Configures the centralized Splunk License Manager.

### `splunk_license_peer`

Configures Splunk Enterprise instances that participate as license peers.

The inventory defines the following components as license peers:

* Cluster Manager
* Indexers
* Search Heads
* Deployer
* Heavy Forwarders
* Deployment Server

### `splunk_ssl`

Configures SSL/TLS for Splunk Web and related Splunk communication.

### `splunk_hosts_file`

Configures `/etc/hosts` entries for the Splunk environment.

This allows the individual Splunk components to resolve internal hostnames consistently.

### `firewall`

Configures the host firewall required for Splunk communication.

### `splunk_all_outputs_idxc_discovery`

Configures Splunk outputs using Indexer Discovery for general Splunk components.

The role is currently used by:

* Heavy Forwarders
* License Manager
* Deployment Server
* Cluster Manager
* Search Head Deployer

### `splunk_shc_outputs_idxc_discovery`

Configures the Search Head Cluster output configuration and distributes the configuration through the Search Head Deployer.

---

# Inventory

The inventory is defined in `inventory.yml`.

The inventory separates hosts into functional Splunk roles rather than treating all machines as identical.

Example:

```yaml
children:

  cluster_manager:
    hosts:
      cm1:

  indexers:
    hosts:
      idx1:
      idx2:
      idx3:

  search_heads:
    hosts:
      sh1:
      sh2:
      sh3:

  deployer:
    hosts:
      dp1:

  deployment_server:
    hosts:
      ds1:

  license_manager:
    hosts:
      lm1:

  heavy_forwarders:
    hosts:
      hf1:
      hf2:

  universal_forwarders:
    hosts:
      uf1:
      uf2:
```

Each host contains both its Ansible connection address and its private IP:

```yaml
idx1:
  ansible_host: CHANGE_ME_PUBLIC_IP
  private_ip: CHANGE_ME_PRIVATE_IP
```

The `private_ip` variable is useful when configuring internal Splunk communication.

---

# Inventory Groups

Two important aggregate groups are defined.

## `license_peers`

The following components participate as license peers:

```text
cluster_manager
indexers
search_heads
deployer
heavy_forwarders
deployment_server
```

This allows license-peer configuration to be applied consistently across the environment.

## `splunk_enterprise`

The `splunk_enterprise` group contains:

```text
cluster_manager
indexers
search_heads
deployer
heavy_forwarders
deployment_server
license_manager
```

This group is used for common Splunk Enterprise operations such as:

* installation
* `/etc/hosts` configuration
* SSL configuration

Universal Forwarders are intentionally separate.

---

# Configuration

## Global Variables

Global configuration is stored in:

```text
group_vars/all.yml
```

An example configuration is provided in:

```text
group_vars/all.yml.example
```

The example file should be used as the starting point for creating a local configuration.

Important variables include:

```yaml
splunk_version: "9.4.10"

splunk_package: "splunk-9.4.10-3673ab0c12ee-linux-amd64.tgz"

splunk_download_location: "/tmp"

splunk_home: "/opt/splunk"

splunk_user: "splunk"
splunk_group: "splunk"

splunk_admin_user: "admin"
```

---

# Secrets

Sensitive values should not be committed to Git.

The project contains variables for:

```yaml
splunk_admin_password:
splunk_shc_secret:
splunk_idxc_secret:
splunk_idxc_discovery_secret:
```

The example configuration intentionally contains:

```text
CHANGE_ME_USE_ANSIBLE_VAULT
```

These values should be replaced using Ansible Vault.

For example:

```bash
ansible-vault encrypt group_vars/all.yml
```

Alternatively, sensitive variables can be separated into an encrypted variables file.

Never commit real:

* Splunk administrator passwords
* cluster secrets
* Indexer Discovery secrets
* private keys
* TLS credentials

to the repository.

---

# Deployment

The entire environment is orchestrated through:

```text
site.yml
```

The playbook is intentionally divided into separate stages.

## Deployment Sequence

The current deployment sequence is:

```text
1.  Install Splunk Enterprise
2.  Configure /etc/hosts
3.  Configure Splunk Web SSL
4.  Configure Firewall
5.  Configure License Manager
6.  Configure License Peers
7.  Configure Cluster Manager
8.  Configure Indexer Cluster
9.  Configure Search Head Deployer
10. Configure Search Head Cluster
11. Connect Search Heads to Indexer Cluster
12. Configure Indexer Discovery
13. Configure Deployment Server
14. Configure SHC Outputs
15. Configure Heavy Forwarder Receiving
16. Configure Heavy Forwarder Outputs
17. Configure License Manager Outputs
18. Configure Deployment Server Outputs
19. Configure Cluster Manager Outputs
20. Configure Deployer Outputs
21. Install Universal Forwarders
```

This ordering establishes the infrastructure from the underlying Splunk installation through clustering and finally to forwarding.

---

# Running the Playbook

After configuring the inventory and variables, test Ansible connectivity:

```bash
ansible all -i inventory.yml -m ping
```

Run the complete deployment:

```bash
ansible-playbook -i inventory.yml site.yml
```

For a dry run:

```bash
ansible-playbook \
  -i inventory.yml \
  site.yml \
  --check
```

To obtain more detailed output:

```bash
ansible-playbook \
  -i inventory.yml \
  site.yml \
  -vv
```

---

# Example Configuration Workflow

Clone the repository:

```bash
git clone https://github.com/michaelsayala/ansible-splunk-cluster.git
cd ansible-splunk-cluster
```

Create your inventory:

```bash
cp inventory.yml.example inventory.yml
```

Create your variables:

```bash
cp group_vars/all.yml.example group_vars/all.yml
```

Update:

```text
ansible_user
ansible_ssh_private_key_file
ansible_host
private_ip
```

Configure the Splunk installation and authentication variables.

Protect sensitive values using Ansible Vault.

Test connectivity:

```bash
ansible all -i inventory.yml -m ping
```

Run the deployment:

```bash
ansible-playbook -i inventory.yml site.yml
```

---

# Verification

After deployment, verify Splunk on each host:

```bash
/opt/splunk/bin/splunk status
```

Check the Splunk management port:

```bash
ss -lntp | grep 8089
```

Check Splunk Web:

```bash
ss -lntp | grep 8000
```

Check the Splunk service:

```bash
systemctl status Splunkd
```

Additional validation should include:

* Cluster Manager health
* Indexer Cluster health
* Search Head Cluster health
* License Manager connectivity
* Deployment Server connectivity
* Universal Forwarder registration
* Indexer Discovery
* Forwarder-to-indexer connectivity
* TLS certificate validation
* Firewall connectivity

---

# Security Considerations

This project automates infrastructure that contains sensitive Splunk management interfaces.

The following practices are recommended:

### Protect credentials

Use Ansible Vault for:

```text
Splunk administrator credentials
Indexer Cluster secrets
Search Head Cluster secrets
Indexer Discovery secrets
TLS private keys
```

### Restrict port 8089

Splunk's management/API port should be restricted to trusted management networks and Splunk components.

Avoid exposing it directly to the Internet.

### Use private networking

Where possible, Splunk-to-Splunk communication should occur over private IP addresses.

### Restrict firewall rules

Only allow the ports required by each Splunk component.

AWS Security Groups and host-level firewalls should be configured consistently.

---

# Design Principles

This project follows several infrastructure-as-code principles.

## Modular Roles

Each Splunk function is implemented as an independent Ansible role.

This makes individual components easier to:

* maintain
* test
* reuse
* troubleshoot
* deploy independently

## Separation of Responsibilities

Splunk installation, clustering, SSL, firewall configuration, receiving, and outputs are separated into individual roles.

## Inventory-Driven Architecture

The inventory defines the topology of the Splunk environment.

Roles use inventory groups rather than hard-coded hostnames wherever possible.

## Dynamic Indexer Discovery

The project uses Indexer Discovery rather than relying exclusively on static indexer addresses.

## Repeatable Deployment

The goal is to make the entire Splunk environment reproducible from Ansible configuration rather than requiring manual configuration of every Splunk node.

---

# Project Status

This project is actively being developed as an infrastructure-as-code implementation for a distributed Splunk Enterprise environment.

Current automation covers:

* Splunk Enterprise installation
* Universal Forwarder installation
* Common Splunk configuration
* Indexer Cluster
* Cluster Manager
* Search Head Cluster
* Search Head Deployer
* Deployment Server
* License Manager
* License Peers
* Heavy Forwarders
* Indexer Discovery
* Outputs configuration
* SSL/TLS
* Host resolution
* Firewall configuration

---

# Future Improvements

Potential improvements include:

* Automated TLS certificate generation and renewal
* Ansible Vault integration for all secrets
* Ansible linting
* Molecule testing
* CI/CD validation through GitHub Actions
* Automated Splunk cluster bootstrap validation
* Automated Search Head Cluster captain validation
* Automated Indexer Cluster health checks
* Post-deployment validation
* AWS dynamic inventory
* More granular firewall rules
* Automated certificate distribution
* Automated Splunk service health validation

---

# License

Add the project's chosen open-source license here.

---

# Author

Michael Ayala

Splunk Engineer | Infrastructure Automation | Cloud | Security

GitHub:

https://github.com/michaelsayala

Repository:

https://github.com/michaelsayala/ansible-splunk-cluster
