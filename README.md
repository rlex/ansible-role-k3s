### ansible-k3s
Fork of [this](https://github.com/rancher/k3s-ansible) role with bunch of modifications

### Description

Compared to original role this one is

* Idempotent and doesn't restart / delete services on each run
* Can mount bpffs (for cilium) and install wireguard
* Have streamlined variable names
* Some other things i probably forgot

Tested on ubuntu 20.04 but should work on any relatively new OS.

### Variables


| Variable name              | Default value  | Description                                                                                                                          |
| -------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| k3s_version                | `v1.18.2+k3s1` | version of k3s to install                                                                                                            |
| k3s_master                 | `false`        | installs k3s server when true                                                                                                        |
| k3s_node                   | `false`        | installs k3s node when true                                                                                                          |
| k3s_master_ip              | see below      | victoriametrics system user                                                                                                          |
| k3s_server_flannel_backend | `vxlan`        | k3s flannel backend to use. Set to none to disable flannel                                                                           |
| k3s_server_disable         | `[]`           | array of k3s packaged components to disable (traefik,metrics-server,etc)                                                             |
| k3s_server_extra_args      | ``             | extra arguments for k3s server ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/server-config/)) |
| k3s_agent_extra_args       | ``             | extra arguments for k3s agent ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/agent-config/))   |
| k3s_bpffs                  | `false`        | mounts /sys/fs/bpf bpffs (needed by cilium)                                                                                          |

### Usage

Your inventory should have k3s_master group (will probably remove that requirement later):

```ini
[k3s_master]
kube-master-1.example.org

[k3s_node]
kube-node-1.example.org
kube-node-2.example.org
```

Example group_vars for k3s_master, switching flannel backend to wireguard (which also causes installation of wireguard package in host operating system), and disabling metrics-server and traefik included in k3s:
```yaml
k3s_managed: true
k3s_master: true
k3s_mount_bpffs: true
k3s_server_flannel_backend: wireguard
k3s_server_disable:
  - metrics-server
  - traefik
```

On k3s_node, just
```yaml
k3s_node: true
```
Is enough

By default master_ip will be ansible_host from node in k3s_master group. In some cases you might want to redefine it (internal network, VPN network, etc)