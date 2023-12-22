### Setting sysctl

Role also allows setting arbitrary sysctl settings using ```k3s_sysctl_config``` variable in dict format:
```yaml
k3s_sysctl_configs:
  fs.inotify.max_user_instances: 128
```
Settings defined with that varible will be persisted in ```/etc/sysctl.d/99-k3s.conf``` file, loading them after system reboots
