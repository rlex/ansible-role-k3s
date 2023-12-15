### k3s and external ip
Sometimes k3s fails to properly detect external and internal ip. For those, you can use ```k3s_external_ip``` and ```k3s_internal_ip``` variables, for example:
Ie:
```yaml
k3s_external_ip: "{{ ansible_default_ipv4['address'] }}"
k3s_internal_ip: "{{ ansible_vpn0.ipv4.address }}"
```
In which case external ip will be ansible default ip and node ip (internal-ip) will be ip address of vpn0 interface
