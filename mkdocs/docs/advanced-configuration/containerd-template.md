### Customizing containerd config template
If you use different version of k3s and/or you want to customize containerd template, you can override path to containerd template with ```k3s_containerd_template``` variable, for example: 
```yaml
k3s_containerd_template: "{{ inventory_dir }}/files/k3s/containerd.toml.tmpl.j2"
```
In that case, role will look for containerd template in ```files/k3s/containerd.toml.tmpl.j2``` inside your inventory folder you defined in ```ansible.cfg```  
