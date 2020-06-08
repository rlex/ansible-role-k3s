### ansible-k3s
Rewrite of [this](https://github.com/rancher/k3s-ansible) role with bunch of stuff.  
I use it for my personal kubernetes test lab running on bunch of cheap 5 EUR KVM VPSes.

### Description

Compared to original role this one is

* Idempotent and doesn't restart / delete services on each run
* Can mount bpffs (for cilium) and install wireguard if needed
* Have streamlined variable names
* Can be used as submodule
* Have UFW management to run it relatively securely on cheap VPSes

Tested on ubuntu 20.04 but should work on any relatively new OS with systemd

### Variables


| Variable name              | Default value  | Description                                                                                                                          |
| -------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| k3s_version                | `v1.18.2+k3s1` | version of k3s to install                                                                                                            |
| k3s_master                 | `false`        | installs k3s server when true                                                                                                        |
| k3s_node                   | `false`        | installs k3s node when true                                                                                                          |
| k3s_master_ip              | see below      | ip of master node                                                                                                                    |
| k3s_server_flannel_backend | `vxlan`        | k3s flannel backend to use. Set to none to disable flannel                                                                           |
| k3s_server_disable         | `[]`           | array of k3s packaged components to disable (traefik,metrics-server,etc)                                                             |
| k3s_server_extra_args      | ``             | extra arguments for k3s server ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/server-config/)) |
| k3s_agent_extra_args       | ``             | extra arguments for k3s agent ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/agent-config/))   |
| k3s_bpffs                  | `false`        | mounts /sys/fs/bpf bpffs (needed by cilium)                                                                                          |
| k3s_node_external_ip       | ``             | specifies k3s external ip                                                                                                            |
| k3s_node_ip                | ``             | specifies k3s node ip                                                                                                                |

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
Is usually enough

By default master_ip will be ansible_host from node in k3s_master group. In some cases you might want to redefine it (internal network, VPN network, etc)

### k3s node and external ip
Sometimes k3s fails to properly detect external and internal ip. For those, you can use this variables:
```
k3s_node_external_ip
k3s_node_ip
```
Ie:
```yaml
k3s_node_external_ip: "{{ ansible_default_ipv4['address'] }}"
k3s_node_ip: "{{ ansible_vpn0.ipv4.address }}"
```
In which case external ip will be ansible default ip and node ip will be ip address of vpn0 interface

### UFW
Cheap VPSes are a great way to start your personal kubernetes dev env / playground / etc.  
For example, contabo.com offers 4 core / 8 gb / 200 gb SSD plan for just 5 EUR/month (probably overselling but who cares at this price).  
SSDnodes.com is another great value hosting provider, offering 4 core / 16 GB ram / 150 GB ssd for $109/year.  
However, unlike other "cloud" providers like vultr, scaleway and etc they lack external firewall, leaving all of your ports exposed by default.  
For example, default flannel settings with VXLAN backend opens VXLAN port on 0.0.0.0, leaving your internal network wide open for anyone curious enough.  
So, if you specify 
```yaml
k3s_ufw_enabled: true
```
role will deploy set of default UFW rules, exposing ports 22, 6443, 80, 443, 51820 on master node and 22, 51820 on worker nodes.  
It will also allow everything on 10.0.0.0/8, allowing internal routing to pass without any issue.  
This rules, of course, can be customized:
```yaml
k3s_master_allow_ports:
  - 22
  - 6443
  - 80
  - 443
  - 51820
k3s_master_allow_subnets:
  - 10.0.0.0/8
k3s_node_allow_ports:
  - 22
  - 51820
k3s_node_allow_subnets:
  - 10.0.0.0/8
```
Please note that it exposes both TCP and UDP ports. You can also specify ranges like "30000:33000"  
This default rules are designed to work with some kind of internal VPN (i use tinc), flannel with wireguard backend and nginx ingress controller on master node.  
Please also note that ufw **will not** prevent access to services exposed via hostPort/nodePort, so pay attention to what you expose.
