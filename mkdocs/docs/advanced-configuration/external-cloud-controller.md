### Provisioning cluster using external cloud-controller-manager
By default, cluster will be installed with k3s "dummy" cloud controller manager. If you deploy your k3s cluster on supported cloud platform (for example hetzner with their [ccm](https://github.com/hetznercloud/hcloud-cloud-controller-manager)) you will need to specify following parameters **before** first cluster start, since cloud controller can't be changed after cluster deployment:

```yaml
k3s_master_additional_config:
  disable-cloud-controller: true
k3s_kubelet_additional_config:
  - "cloud-provider=external"
```
