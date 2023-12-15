### Setting sysctl

Role also allows setting arbitrary sysctl settings using ```k3s_sysctl_config``` variable in dict format:
```yaml
k3s_sysctl_configs:
  fs.inotify.max_user_instances: 128
```
Settings defined with that varible will be persisted in ```/etc/sysctl.d/99-k3s.conf``` file, loading them after system reboots

### Setting kubelet arguments
To pass arguments for kubelet, you can use ```k3s_kubelet_additional_config``` variable:
```yaml
k3s_kubelet_additional_config:
  - "image-gc-high-threshold=40"
  - "image-gc-low-threshold=30"
```
