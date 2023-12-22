
# Adding custom registries
By using k3s_registries variable you can configure custom registries, both origins and mirrors. Format follows [official](https://rancher.com/docs/k3s/latest/en/installation/private-registry/) config format.
Example:
```yaml
k3s_registries:
  mirrors:
    docker.io:
      endpoint:
        - "https://mycustomreg.com:5000"
  configs:
    "mycustomreg:5000":
      auth:
        username: xxxxxx # this is the registry username
        password: xxxxxx # this is the registry password
      tls:
        cert_file: # path to the cert file used in the registry
        key_file:  # path to the key file used in the registry
        ca_file:   # path to the ca file used in the registry
```
