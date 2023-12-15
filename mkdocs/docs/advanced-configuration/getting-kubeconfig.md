
### Getting kubeconfig file via role
Role have ability to download kubeconfig file to machine from where ansible was run. To use it, set following variables:
```yaml
k3s_kubeconfig: true
k3s_kubeconfig_context: k3s-de1
```
Role will perform following:

1. Copy ```/etc/rancher/k3s/k3s.yml``` to ```~/.kube/config-${ k3s_kubeconfig_context``` }

2. Patch it with your preferred context name specified in ```k3s_kubeconfig_context``` variable instead of stock ```default```

3. Patch it with proper server URL (by default, it will be ansible_host of first master node in group specified in variable ```k3s_master_group```, with port 6443, aka "initial master"), but you can override it with ```k3s_kubeconfig_server```

4. Download resulting file to machine running ansible with path ```~/.kube/config-${ k3s_kubeconfig_context }```, in current example it will be ```~/.kube/config-k3s-de1```

And you can start using it right away.
However, if your master is configured differently (HA IP, Load balancer, etc), you might want to specify server manually. For this, you can use ```k3s_kubeconfig_server``` variable:
```yaml
k3s_kubeconfig_server: "master-ha.k8s.example.org:6443"
```
Please note that role *will not* track changes of ```/etc/rancher/k3s/k3s.yml``` - if you redeploy your k3s cluster and need new kubeconfig, just delete existing local kubeconfig to get new one.
