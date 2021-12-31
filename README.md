### ansible-k3s
Ansible role for managing rancher [k3s](https://k3s.io), lightweight, cncf-certified kubernetes distribution.  
I use it for my personal kubernetes test lab running on bunch of cheap KVM VPSes.  
It's tailored for my needs, but it's still pretty generic and can be used anywhere.  

### Description
Compared to original role by rancher this one is
* Idempotent and doesn't restart / delete services on each run
* Can mount bpffs (for cilium/etc) and install wireguard if needed
* Have streamlined variable names
* Can install multi-master configuration using builtin etcd server

Tested on ubuntu 20.04 but should work on any relatively new OS - all it requires is systemd

### Variables

| Variable name                | Default value  | Description                                                                                                                          |
| ---------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| k3s_version                  | `v1.22.3+k3s1` | version of k3s to install                                                                                                            |
| k3s_master                   | `false`        | installs k3s master when true                                                                                                        |
| k3s_agent                    | `false`        | installs k3s agent when true                                                                                                         |
| k3s_master_ip                | see below      | ip of master node                                                                                                                    |
| k3s_master_port              | `6443`         | port of masterserver                                                                                                                 |
| k3s_flannel_backend          | `vxlan`        | k3s flannel backend to use. Set to none to disable flannel                                                                           |
| k3s_server_disable           | `[]`           | array of k3s packaged components to disable (traefik,metrics-server,etc)                                                             |
| k3s_master_extra_args        | `[]`           | extra arguments for k3s server ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/server-config/)) |
| k3s_master_additional_config | ``             | YAML with extra config for k3s master                                                                                                |
| k3s_agent_additional_config  | ``             | YAML with extra config for k3s agent                                                                                                 |
| k3s_agent_extra_args         | `[]`           | extra arguments for k3s agent ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/agent-config/))   |
| k3s_bpffs                    | `false`        | mounts /sys/fs/bpf bpffs (needed by some network stacks)                                                                             |
| k3s_external_ip              | ``             | specifies k3s external ip                                                                                                            |
| k3s_internal_ip              | ``             | specifies k3s node ip                                                                                                                |
| k3s_registries               | ``             | Configures custom registries, see [official docs](https://rancher.com/docs/k3s/latest/en/installation/private-registry/) for format  |
| k3s_gvisor                   | ``             | Installs [gvisor](https://gvisor.dev)                                                                                                |
| k3s_agent_group              | k3s_node       | specifies ansible group name for k3s nodes                                                                                           |
| k3s_master_group             | k3s_master     | specifies ansible group name for k3s master(s)                                                                                       |
| k3s_additional_packages      | `[]`           | Installs additional packages if needed by workloads (ie iscsid)                                                                      |
| k3s_additional_services      | `[]`           | Enables additional services if needed by workloads (ie iscsid)                                                                       |

### Usage

By default, your inventory should have k3s_master and k3s_agent groups:
```ini
[k3s_master]
kube-master-1.example.org

[k3s_agent]
kube-node-1.example.org
kube-node-2.example.org
```

You can always change this group names using k3s_node_group and k3s_master_group variables, for example if you have multiple k3s clusters in single inventory file or just want different group names.  
Example group_vars for k3s_master, switching flannel backend to wireguard (which also causes installation of wireguard package in host operating system), and disabling metrics-server and traefik included in k3s:
```yaml
k3s_master: true
k3s_mount_bpffs: true
k3s_flannel_backend: wireguard
k3s_server_disable:
  - metrics-server
  - traefik
```

On k3s_node, just
```yaml
k3s_agent: true
```
Is usually enough

By default k3s_master_ip will be set to ansible_host from node in k3s_master group.  
In case of multiple masters, it will be set to ansible_host of first node in k3s_master group.  
In some cases you might want to redefine it (internal network, VPN network, etc). For that, you can use k3s_master_ip variable.

### Multi-master setup
When your k3s_master ansible inventory group have more than 1 host, role will detect it and switch to multi-master installation.  
First node in group will be used as "bootstrap", while following will bootstrap from first node

You can also switch existing, single-node sqlite master to multimaster configuration by adding more masters to existing install - be aware, however, that migration from single-node sqlite to etcd is supported only in k3s >= 1.22!

Pay attention, however, that in default configuration all agents will be pointing only to first master, which is not really useful for HA setup. Configuring HA is out of scope for this role, but luckily there is some really good ansible roles available for that:

1) [This keepalived role](https://github.com/Oefenweb/ansible-keepalived) if you have L2 networking available and can use VRRP for failover IP
2) [This haproxy role](https://github.com/Oefenweb/ansible-haproxy). I run my cluster on top of L3 vpn so i can't use L2, so i just install haproxy on each agent node, point haproxy to all masters, and point agents to localhost haproxy. Dirty, but works.

### k3s and external ip
Sometimes k3s fails to properly detect external and internal ip. For those, you can use this variables:
```
k3s_external_ip
k3s_internal_ip
```
Ie:
```yaml
k3s_external_ip: "{{ ansible_default_ipv4['address'] }}"
k3s_internal_ip: "{{ ansible_vpn0.ipv4.address }}"
```
In which case external ip will be ansible default ip and node ip will be ip address of vpn0 interface

### Additional packages and services
Sometimes certain software requires some packages installed on host system. One of examples is distributed filesystems like longhorn and openebs which require iscsid
While it's better to manage such software with dedicated roles, i included that variables for simplicity. If you want openebs-jiva or longhorn to work, you can specify
```yaml
k3s_additional_packages:
  - open-iscsi
k3s_additional_services:
  - iscsid
```
open-iscsi will be installed and iscsid service will be both started and enabled at boot-time before k3s installation

### Custom network plugin
If you want to use something different than default flannel you can set flannel backend to none, which will remove flannel
```yaml
k3s_flannel_backend: none
```
Additionally, if you want to use something with eBPF dataplane enabled (calico, cilium) you might need to disable kube-proxy and mount bpffs filesystem on host node:
```yaml
k3s_bpffs: true
k3s_master_additional_config:
  disable-kube-proxy: true
```

### Custom registries
By using k3s_registries variable you can configure custom registries, both origins and mirrors. Format follows [official](https://rancher.com/docs/k3s/latest/en/installation/private-registry/) config format.  
Example:
```yaml
k3s_registries:
  mirrors:
    docker.io:
      endpoint:
        - "https://mycustomreg.com:5000"
  configs:
    "mycustomreg:5000":
      auth:
        username: xxxxxx # this is the registry username
        password: xxxxxx # this is the registry password
      tls:
        cert_file: # path to the cert file used in the registry
        key_file:  # path to the key file used in the registry
        ca_file:   # path to the ca file used in the registry
```

### Gvisor

By setting k3s_gvisor to true role will install gvisor - google's application kernel for container. By default it will use ptrace mode.  
It will only install gvisor containerd plugin, you will need to create gvisor's RuntimeClass manually by applying following manifest:  
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```
After that you should be able to launch gvisor-enabled pods by adding runtimeClassName to pod spec, ie
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gvisor-ubuntu
spec:
  runtimeClassName: gvisor
  containers:
    - name: nginx
      image: nginx
```
