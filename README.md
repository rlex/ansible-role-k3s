 <!-- TOC -->

- [Description](#description)
- [Requirements](#requirements)
- [Variables](#variables)
- [Usage](#usage)
- [Multi-master setup](#multi-master-setup)
  - [HA with haproxy](#ha-with-haproxy)
  - [HA with VRRP (keepalived)](#ha-with-vrrp-keepalived)
- [k3s and external ip](#k3s-and-external-ip)
- [Getting kubeconfig file via role](#getting-kubeconfig-file-via-role)
- [Additional packages and services](#additional-packages-and-services)
- [Using custom network plugin](#using-custom-network-plugin)
- [Adding custom registries](#adding-custom-registries)
- [Setting sysctl](#setting-sysctl)
- [Setting kubelet arguments](#setting-kubelet-arguments)
- [Creating additional configs](#creating-additional-configs)
- [Provisioning cluster using external cloud-controller-manager](#provisioning-cluster-using-external-cloud-controller-manager)
- [Sandboxing workloads with gvisor](#sandboxing-workloads-with-gvisor)
  - [Warning](#warning)
  - [Usage](#usage-1)
  - [Additional configuration](#additional-configuration)
- [To be done / some ideas](#to-be-done--some-ideas)

<!-- /TOC -->

### Description
Ansible role for managing rancher [k3s](https://k3s.io), lightweight, cncf-certified kubernetes distribution.  
I use it for my personal kubernetes installs/test labs running on bunch of cheap KVM VPSes, some raspberries and some cloud VMs and so on.  
It's tailored for my needs (ie gvisor and etc), but it's still very generic and can be used anywhere, plus all of my custom stuff can be disabled via variables

### Requirements
Apart from [what k3s requires](https://rancher.com/docs/k3s/latest/en/installation/installation-requirements/), this role also needs systemd.

### Variables

| Variable name                 | Default value                    | Description                                                                                                                          |
| ----------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| k3s_version                   | `v1.24.8+k3s1`                   | version of k3s to install                                                                                                            |
| k3s_master                    | `false`                          | installs k3s master when true                                                                                                        |
| k3s_agent                     | `false`                          | installs k3s agent when true                                                                                                         |
| k3s_master_ip                 | see below                        | ip of master node                                                                                                                    |
| k3s_master_port               | `6443`                           | port of masterserver                                                                                                                 |
| k3s_flannel_backend           | `vxlan`                          | k3s flannel backend to use. Set to none to disable flannel                                                                           |
| k3s_server_disable            | `[]`                             | array of k3s packaged components to disable (traefik,metrics-server,etc)                                                             |
| k3s_master_extra_args         | `[]`                             | extra arguments for k3s server ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/server-config/)) |
| k3s_master_additional_config  | ``                               | YAML with extra config for k3s master                                                                                                |
| k3s_agent_additional_config   | ``                               | YAML with extra config for k3s agent                                                                                                 |
| k3s_kubelet_additional_config | ``                               | Additional arguments for kubelet, see docs                                                                                           |
| k3s_agent_extra_args          | `[]`                             | extra arguments for k3s agent ([official docs](https://rancher.com/docs/k3s/latest/en/installation/install-options/agent-config/))   |
| k3s_additional_config_files   | `{}`                             | [extra configfiles for k3s](#creating-additional-configs)                                                                            |
| k3s_bpffs                     | `false`                          | mounts /sys/fs/bpf bpffs (needed by some network stacks)                                                                             |
| k3s_external_ip               | ``                               | specifies k3s external ip                                                                                                            |
| k3s_internal_ip               | ``                               | specifies k3s node ip                                                                                                                |
| k3s_registries                | ``                               | Configures custom registries, see [official docs](https://rancher.com/docs/k3s/latest/en/installation/private-registry/) for format  |
| k3s_cronjob_prune_images      | `absent`                         | Configures cronjob that prunes unused images in containerd daily. Either `absent` or `present`                                       |
| k3s_gvisor                    | `false`                          | Installs [gvisor](https://gvisor.dev)                                                                                                |
| k3s_gvisor_platform           | `systrap`                        | Selects [platform](https://gvisor.dev/docs/architecture_guide/platforms/) to use in gvisor                                           |
| k3s_gvisor_config             | ``                               | Sets additional options for gvisor runsc. See notes                                                                                  |
| k3s_kubeconfig                | false                            | Downloads kubeconfig to machine from which role was launched                                                                         |
| k3s_kubeconfig_server         | see below                        | specifies server for use in kubeconfig                                                                                               |
| k3s_kubeconfig_context        | k3s                              | specifies context to use in kubeconfig                                                                                               |
| k3s_kubeconfig_target:        | ``{{ k3s_kubeconfig_context }}`` | specifies filename for downloading kubeconfig                                                                                        |
| k3s_agent_group               | k3s_node                         | specifies ansible group name for k3s nodes                                                                                           |
| k3s_master_group              | k3s_master                       | specifies ansible group name for k3s master(s)                                                                                       |
| k3s_additional_packages       | `[]`                             | Installs additional packages if needed by workloads (ie iscsid)                                                                      |
| k3s_additional_services       | `[]`                             | Enables additional services if needed by workloads (ie iscsid)                                                                       |
| k3s_sysctl_config             | `{}`                             | Allows setting arbitrary sysctl settings                                                                                             |

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

On k3s_agent, just
```yaml
k3s_agent: true
```
Is usually enough.  
By default k3s_master_ip will be set to ansible_host from node in k3s_master group.  
In case of multiple masters, it will be set to ansible_host of first node in k3s_master group.  
In some cases you might want to redefine it (internal network, VPN network, etc). For that, you can use k3s_master_ip variable.

### Multi-master setup
When your k3s_master ansible inventory group have more than 1 host, role will detect it and switch to multi-master installation.  
First node in group will be used as "bootstrap", while following will bootstrap from first node

You can also switch existing, single-node sqlite master to multimaster configuration by adding more masters to existing install - be aware, however, that migration from single-node sqlite to etcd is supported only in k3s >= 1.22!

Pay attention, however, that in default configuration all agents will be pointing only to first master, which is not really useful for HA setup. Configuring HA is out of scope for this role, but i'll cover some ideas of HA setup:

#### HA with haproxy
Using [This haproxy role](https://github.com/Oefenweb/ansible-haproxy). I run my cluster on top of L3 vpn so i can't use L2, so i just install haproxy on each node, point haproxy to all masters, and point agents to localhost haproxy. Dirty, but works. Example config:

```yaml
haproxy_listen:
  - name: stats
    description: Global statistics
    bind:
      - listen: '0.0.0.0:1936'
    mode: http
    http_request:
      - action: use-service
        param: prometheus-exporter
        cond: if { path /metrics }
    stats:
      enable: true
      uri: /
      options:
        - hide-version
        - show-node
      admin: if LOCALHOST
      refresh: 5s
      auth:
        - user: admin
          passwd: 'yoursupersecretpassword'
haproxy_frontend:
  - name: kubernetes_master_kube_api
    description: frontend with k8s api masters
    bind:
      - listen: "127.0.0.1:16443"
    mode: tcp
    default_backend: k8s-de1-kube-api
haproxy_backend:
  - name: k8s-de1-kube-api
    description: backend with all kubernetes masters
    mode: tcp
    balance: roundrobin
    option:
      - httpchk GET /readyz
    http_check: expect status 401
    default_server_params:
      - inter 1000
      - rise 2
      - fall 2
    server:
      - name: k8s-de1-master-1
        listen: "master-1:6443"
        param:
          - check
          - check-ssl
          - verify none
      - name: k8s-de1-master-2
        listen: "master-2:6443"
        param:
          - check
          - check-ssl
          - verify none
      - name: k8s-de1-master-3
        listen: "master-3:6443"
        param:
          - check
          - check-ssl
          - verify none
```

That will start haproxy listening on 127.0.0.1:16443 for connections to k8s masters. You can then redefine master IP and port for agents with
```yaml
k3s_master_ip: 127.0.0.1
k3s_master_port: 16443
```

And now your connections are balanced between masters and protected in case of one or two masters will go down. One downside of that config is that it checks for reply 401 on /readyz endpoint, because since certain version of k8s (1.19 if i recall correctly) this endpoint requires authorization. So we just check that it works, not for actual "OK" reply and 200 HTTP code.  
This proxy also works with initial agent join, so it's better to setup haproxy before installing k3s and then switching to HA config.
It will also expose prometheus metrics on 0.0.0.0:1936/metrics - pay attention that this part (unlike webui) won't be protected by user and password, so adjust your firewall accordingly if needed!

Of course you can use whatever you want - external cloud LB, nginx, anything, all it needs is TCP protocol support (because in this case we don't want to manage SSL on loadbalancer side). But haproxy provides you with prometheus metrics, have nice webui for monitoring and management, and i'm just familiar with it.

#### HA with VRRP (keepalived)
You can use [this keepalived role](https://github.com/Oefenweb/ansible-keepalived) if you have L2 networking available and can use VRRP for failover IP. In that case, you might need to add tls-san option in k3s_master_additional_config with your floating ip.
For keepalived to work, following needs should be met:
  1) L2 networking must be available. Sadly, this is not a common case with cloud providers and most VPNs.
  2) Virtual IP must be in same subnet as interfaces on top of which they are used

Sample keepalived configuration on master-1, assuming we use network 10.91.91.0/24 on vpn0 interface:
```yaml
keepalived_instances:
  vpn:
    interface: vpn0
    state: MASTER
    virtual_router_id: 51 #if you have multiple VRRP setups in same network this should be unique
    priority: 255 #node usually owning IP should always have priority set to 255
    authentication_password: "somepassword" #can be omitted, but always good to use
    vips:
      - "10.91.91.50 dev vpn0 label vpn0:0"
```
for backing masters:
```yaml
keepalived_instances:
  vpn:
    interface: vpn0
    state: BACKUP
    virtual_router_id: 51
    priority: 254 #use lower priority for each node
    authentication_password: "somepassword"
    vips:
      - "10.91.91.50 dev vpn0 label vpn0:0"
```
And in k3s configuration:
```yaml
k3s_master_additional_config:
  tls-san: 10.91.91.50
```

If everything is configured correctly, you should see 10.91.91.50 on vpn0:0 interface on master-1 node. Try stopping keepalived on master-1 and see how IP disappears from master-1 and appears on master-2.  
From now it's your choice how you want to configure HA - point agents to that floating IP, or install load-balancer on each master node and distribute requests between them.
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
In which case external ip will be ansible default ip and node ip (internal-ip) will be ip address of vpn0 interface

### Getting kubeconfig file via role
Role have ability to download kubeconfig file to machine from where ansible was run. To use it, set following variables:
```
k3s_kubeconfig: true
k3s_kubeconfig_context: k3s-de1
```
Role will perform following:
1. Copy /etc/rancher/k3s/k3s.yml to ~/.kube/config-${ k3s_kubeconfig_context }
2. Patch it with your preferred context name specified in k3s_kubeconfig_context variable instead of stock "default"
3. Patch it with proper server URL (by default, it will be ansible_host of first master node in group specified in variable k3s_master_group, with port 6443, aka "initial master"), but you can override it with k3s_kubeconfig_server
4. Download resulting file to machine running ansible with path ~/.kube/config-${ k3s_kubeconfig_context }, in current example it will be ~/.kube/config-k3s-de1
And you can start using it right away.
However, if your master is configured differently (HA IP, Load balancer, etc), you might want to specify server manually. For this, you can use k3s_kubeconfig_server variable:
```
k3s_kubeconfig_server: "master-ha.k8s.example.org:6443"
```
Please note that role will not track changes of /etc/rancher/k3s/k3s.yml - if you redeploy your k3s cluster and need new kubeconfig, just delete existing local kubeconfig to get new one.

### Additional packages and services
Sometimes certain software requires certain packages installed on host system. Some of examples are distributed filesystems like longhorn and openebs which require iscsid.  
While it's better to manage such software with dedicated roles, i included that variables for simplicity. If you want openebs-jiva or longhorn to work, you can specify
```yaml
k3s_additional_packages:
  - open-iscsi
k3s_additional_services:
  - iscsid
```
open-iscsi will be installed and iscsid service will be both started and enabled at boot-time before k3s installation

### Using custom network plugin
If you want to use something different and self-managed than default flannel you can set flannel backend to none, which will remove flannel completely:
```yaml
k3s_flannel_backend: none
```
Additionally, if you want to use something with eBPF dataplane enabled (calico, cilium) you might need to disable kube-proxy and mount bpffs filesystem on host node:
```yaml
k3s_bpffs: true
k3s_master_additional_config:
  disable-kube-proxy: true
```

### Adding custom registries
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

### Setting sysctl

Role also allows setting arbitrary sysctl settings using k3s_sysctl_config variable in dict format:
```
k3s_sysctl_configs:
  fs.inotify.max_user_instances: 128
```
Settings defined with that varible will be persisted in /etc/sysctl.d/99-k3s.conf file, loading them after system reboots

### Setting kubelet arguments
To pass arguments for kubelet, you can use k3s_kubelet_additional_config variable:
```
k3s_kubelet_additional_config:
  - "image-gc-high-threshold=40"
  - "image-gc-low-threshold=30"
```

### Creating additional configs
Sometimes you need to create additional config files. For example, you want to trace kubelet, which requires separate config file for tracing configuration.  
Variable k3s_additional_config_files will take care of that.  
All additional config files will go to /etc/rancher/k3s directory, with name specified in name block and with content specified in content.  
This action will happen on pre-configuration stage, before k3s installation.  

Example:
```yaml
k3s_additional_configfiles:
  - name: apiserver-tracing.yaml
    content: |
      apiVersion: apiserver.config.k8s.io/v1alpha1
      kind: TracingConfiguration
      endpoint: 127.0.0.1:4317
      samplingRatePerMillion: 100
```

Will result in file /etc/rancher/k3s/apiserver-tracing.yaml with following content:
```yaml
apiVersion: apiserver.config.k8s.io/v1alpha1
kind: TracingConfiguration
endpoint: 127.0.0.1:4317
samplingRatePerMillion: 100
```

Please note that no additional formatting or processing is happening on that stage, so you need to take care about all indentation and other formatting things.  
Additionally, editing any of those files will trigger k3s restart

### Provisioning cluster using external cloud-controller-manager
By default, cluster will be installed with k3s "dummy" cloud controller manager. If you deploy your k3s cluster on supported cloud platform (for example hetzner with their [ccm](https://github.com/hetznercloud/hcloud-cloud-controller-manager)) you will need to specify following parameters **before** first cluster start, since cloud controller can't be changed after cluster deployment:

```
k3s_master_additional_config:
  disable-cloud-controller: true

k3s_kubelet_additional_config:
  - "cloud-provider=external"
```

### Sandboxing workloads with gvisor

#### Warning
Due to how k3s works, you might need to sync [k3s built-in containerd config](https://github.com/k3s-io/k3s/blob/v1.25.2%2Bk3s1/pkg/agent/templates/templates_linux.go) according to k3s version you use. Config in this repo is synced with version specified in k3s_version variable.

If you decide to customize containerd config, you will need to verify there is no double configuration present in containerd config - for example, [gvisor containerd guide](https://gvisor.dev/docs/user_guide/containerd/configuration/) have following block:
```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"
```
which is already present in basic k3s containerd config

#### Usage
By setting k3s_gvisor to true role will install gvisor - google's application kernel for container. By default it will use systrap mode, to switch it to kvm set k3s_gvisor_platform to kvm.  
It will only install gvisor containerd plugin, you will need to create gvisor's RuntimeClass manually by applying following manifest for gvisor:
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
  name: gvisor-nginx
spec:
  runtimeClassName: gvisor
  containers:
    - name: nginx
      image: nginx
```
#### Additional configuration
Role supports passing additional settings for gvisor using k3s_gvisor_config. For example, to enable host networking, use:
```yaml
k3s_gvisor_config:
  network: host
```
Which will become
```toml
[runsc_config]
  network = "host"
```
in gvisor config

### To be done / some ideas
  * Additional configuration for gvisor
  * In case of gvisor is running in kvm mode, load corresponding kernel modules
  * Automatic selinux policy installer for centos/etc
  * Maybe some basic management of k8s resources (ie creating gvisor RuntimeClass if gvisor is enabled)
