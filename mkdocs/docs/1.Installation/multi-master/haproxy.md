#### HA with haproxy
Using [This haproxy role](https://github.com/Oefenweb/ansible-haproxy). I run my cluster on top of L3 vpn so i can't use L2, so i just install haproxy on each node, point haproxy to all masters, and point agents to localhost haproxy. Dirty, but works. Example config:

```yaml
haproxy_listen:
  - name: stats
    description: Global statistics
    bind:
      - listen: '0.0.0.0:1936'
    mode: http
    http_request:
      - action: use-service
        param: prometheus-exporter
        cond: if { path /metrics }
    stats:
      enable: true
      uri: /
      options:
        - hide-version
        - show-node
      admin: if LOCALHOST
      refresh: 5s
      auth:
        - user: admin
          passwd: 'yoursupersecretpassword'
haproxy_frontend:
  - name: kubernetes_master_kube_api
    description: frontend with k8s api masters
    bind:
      - listen: "127.0.0.1:16443"
    mode: tcp
    default_backend: k8s-de1-kube-api
haproxy_backend:
  - name: k8s-de1-kube-api
    description: backend with all kubernetes masters
    mode: tcp
    balance: roundrobin
    option:
      - httpchk GET /readyz
    http_check: expect status 401
    default_server_params:
      - inter 1000
      - rise 2
      - fall 2
    server:
      - name: k8s-de1-master-1
        listen: "master-1:6443"
        param:
          - check
          - check-ssl
          - verify none
      - name: k8s-de1-master-2
        listen: "master-2:6443"
        param:
          - check
          - check-ssl
          - verify none
      - name: k8s-de1-master-3
        listen: "master-3:6443"
        param:
          - check
          - check-ssl
          - verify none
```

That will start haproxy listening on 127.0.0.1:16443 for connections to k8s masters. You can then redefine master IP and port for agents with
```yaml
k3s_master_ip: 127.0.0.1
k3s_master_port: 16443
```

And now your connections are balanced between masters and protected in case of one or two masters will go down. One downside of that config is that it checks for reply 401 on /readyz endpoint, because since certain version of k8s (1.19 if i recall correctly) this endpoint requires authorization. So you have 2 options here:

  * Continue to rely on 401 check (not a good solution, since we're just checking for http up status)
  * Add ```anonymous-auth=true``` to apiserver arguments: 
      ```yaml
        k3s_master_additional_config:
          kube-apiserver-arg:
          - "anonymous-auth=true"
      ```
    This will open /readyz, /healthz, /livez and /version endpoints to anonymous auth, and potentially expose version info. If that is concerning you, it's possible to patch system:public-info-viewer role to keep only /readyz, /healthz and /livez endpoint open:
    ```
    kubectl patch clusterrole system:public-info-viewer --type=json -p='[{"op": "replace", "path": "/rules/0/nonResourceURLs", "value":["/healthz","/livez","/readyz"]}]'
    ```
  
This proxy also works with initial agent join, so it's better to setup haproxy before installing k3s and then switching to HA config.
It will also expose prometheus metrics on 0.0.0.0:1936/metrics - pay attention that this part (unlike webui) won't be protected by user and password, so adjust your firewall accordingly if needed!

Of course you can use whatever you want - external cloud LB, nginx, anything, all it needs is TCP protocol support (because in this case we don't want to manage SSL on loadbalancer side). But haproxy provides you with prometheus metrics, have nice webui for monitoring and management, and i'm just familiar with it.
