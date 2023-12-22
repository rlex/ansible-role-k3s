# HA with VRRP (keepalived)
You can use [this keepalived role](https://github.com/Oefenweb/ansible-keepalived) if you have L2 networking available and can use VRRP for failover IP. In that case, you might need to add tls-san option in k3s_master_extra_config with your floating ip.
For keepalived to work, following conditions should be met:
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
k3s_master_extra_config:
  tls-san: 10.91.91.50
```

If everything is configured correctly, you should see 10.91.91.50 on vpn0:0 interface on master-1 node. Try stopping keepalived on master-1 and see how IP disappears from master-1 and appears on master-2.  
From now it's your choice how you want to configure HA - point agents to that floating IP, or install load-balancer on each master node and distribute requests between them.
