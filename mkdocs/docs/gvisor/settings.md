#### Additional configuration
Role supports passing additional settings for gvisor using ```k3s_gvisor_config```. For example, to enable host networking, use:
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
