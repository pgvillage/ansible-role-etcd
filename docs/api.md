# pgvillage.etcd API

This document describes all variables that can be used to configure the `pgvillage.etcd` role.
Defaults are defined in [defaults/main.yml](../defaults/main.yml).

## Installation

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_package_version` | `present` | Package state passed to `ansible.builtin.package` (e.g. `present`, `latest`). |
| `etcd_package_name` | `etcd` | Name of the etcd package to install. |
| `etcd_install_dir` | `/usr/local/bin` | Directory containing the etcd binary (used in the systemd unit `ExecStart`). |

## User and directories

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_user` | `etcd` | OS user that owns and runs etcd. |
| `etcd_group` | `etcd` | OS group of the etcd user. |
| `etcd_data_dir` | `/var/lib/etcd` | Base data directory (also the etcd user's home). Cluster data and pki directories are created below it as `<etcd_cluster_name>.etcd` and `<etcd_cluster_name>.pki`. |

## Cluster

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_master_group_name` | `etcd_master` | Inventory group with the voting etcd members. Hosts not in this group run as etcd proxy. |
| `etcd_cluster` | *(generated)* | Value for `ETCD_INITIAL_CLUSTER`. By default built from all hosts in `etcd_master_group_name` as `<fqdn>=<scheme><cluster address>:<etcd_port_peer>`, comma separated. |
| `etcd_cluster_name` | `test-cluster-name` | Cluster name, used to name the data and pki directories below `etcd_data_dir`. |
| `etcd_initial_cluster_token` | `d8bf8cc6-5158-11e6-8f13-3b32f4935bde` | Initial cluster token (`ETCD_INITIAL_CLUSTER_TOKEN`); should be unique per cluster. |

## Networking

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_use_ips` | `true` | Advertise IP addresses (`true`) or the host FQDN (`false`) for client and peer URLs. |
| `etcd_network_iface` | *(undefined)* | Optional. When set, used as default for both `etcd_iface_public` and `etcd_iface_cluster`. |
| `etcd_iface_public` | `{{ etcd_network_iface \| default("all") }}` | Interface for client traffic: `all` (listen on `0.0.0.0`), `default` (default IPv4 interface), or an interface name (e.g. `eth1`). When not `all`, etcd also listens on `127.0.0.1`. |
| `etcd_iface_cluster` | `{{ etcd_network_iface \| default("default") }}` | Interface for peer (cluster) traffic: `all`, `default` or an interface name. |
| `etcd_port_client` | `2379` | Port for client requests. |
| `etcd_port_peer` | `2380` | Port for peer (cluster) communication. |

## Security (TLS)

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_secure` | `false` | Enable TLS (https) for client and peer communication, with client cert authentication. |
| `etcd_pki_dir` | `~/pki-dir` | Directory on the Ansible controller holding the keys/certs to deploy when `etcd_secure` is true. |
| `etcd_pki_key_suffix` | `-key.pem` | Filename suffix of private key files in `etcd_pki_dir`. |
| `etcd_pki_cert_suffix` | `.pem` | Filename suffix of certificate files (host and CA) in `etcd_pki_dir`. |

When `etcd_secure` is enabled, the following files are expected in `etcd_pki_dir` for every host:

- `<inventory_hostname><etcd_pki_key_suffix>` — host private key
- `<inventory_hostname><etcd_pki_cert_suffix>` — host certificate
- `ca<etcd_pki_cert_suffix>` — CA certificate

They are copied to `<etcd_data_dir>/<etcd_cluster_name>.pki` on the target hosts.

## Service

| Variable | Default | Description |
|----------|---------|-------------|
| `etcd_init_system` | `systemd` | Init system to configure; selects `tasks/<etcd_init_system>.yml`. Only `systemd` is supported. |
| `etcd_launch` | `true` | Enable and start etcd (and allow handlers to restart it). |
| `etcd_enable_v2` | `true` | Accept etcd V2 client requests (`ETCD_ENABLE_V2`). |
| `etcd_additional_envvars` | `{}` | Extra environment variables written to `/etc/etcd/etcd.conf`, e.g. `{ETCD_HEARTBEAT_INTERVAL: "100"}`. |

## Example

```yaml
- hosts: etcd
  vars:
    etcd_cluster_name: pgv-prod
    etcd_initial_cluster_token: 3f6c1e2a-0b7d-4c8e-9a41-5d2f7e8b6c10
    etcd_secure: true
    etcd_pki_dir: ./pki
    etcd_iface_cluster: eth1
  roles:
    - pgvillage.etcd
```
