### Adding custom kubernetes manifests

If you need to create additional kubernetes objects after cluster creation, you can use k3s_additional_manifests variable.<br> 
Example with all possible parameters:

```yaml
k3s_additional_manifests:
  - name: kata
    state: present
    definition:
      apiVersion: node.k8s.io/v1
      kind: RuntimeClass
      metadata:
        name: kata
      handler: kata
```

You can supply full definition in "definition" block, including resource name in metadata.name (gvisor-test1 in example).<br>
If your object doesn't contain metadata.name, then name from ansible will be used (gvisor-test12 in example).<br>
Object name in .definition have precedence and will be used if both .name and .definition.metadata.name exists.<br>
You can also control control resource state with state parameter (absent, present), which is set to "present" by default.<br> 
Object creation will be delegated to first node in your k3s_master_group, in case of multi-master setup it will be your "initial" master node.<br>
For RBAC, it will use k3s-generated /etc/rancher/k3s/k3s.yaml kubeconfig on same master server, which have cluster-admin rights.<br>
