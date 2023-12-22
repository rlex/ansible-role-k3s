### Multi-master setup
When your ```k3s_master``` ansible inventory group have more than 1 host, role will detect it and switch to multi-master installation.  
First node in group will be used as "bootstrap", while following will bootstrap from first node.  
Any number of nodes is ok, but it's generally recommended to have odd number of nodes for etcd eletction to work, since etcd quorum is (n/2)+1 where n is number of nodes.  

You can also switch existing, single-node sqlite master to multimaster configuration by adding more masters to existing install - be aware, however, that migration from single-node sqlite to etcd is supported only in k3s >= 1.22!

Pay attention that in default configuration all agents will be pointing only to first master, which is not really useful for HA setup. Configuring HA is out of scope for this role, so take a look at following docs:

1. [HA with haproxy](high-availability/haproxy.md)
2. [HA with keepalived](high-availability/keepalived.md)

