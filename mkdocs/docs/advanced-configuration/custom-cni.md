# Using custom network plugin
If you want to use something different and self-managed than default flannel you can set flannel backend to none, which will remove flannel completely:
```yaml
k3s_flannel_backend: none
```
Additionally, if you want to use something with eBPF dataplane enabled (calico, cilium) you might need to disable kube-proxy and mount bpffs filesystem on host node:
```yaml
k3s_bpffs: true
k3s_master_extra_config:
  disable-kube-proxy: true
```
