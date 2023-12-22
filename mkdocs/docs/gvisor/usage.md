# Installation and usage
By setting k3s_gvisor to true role will install gvisor - google's application kernel for container.<br> 
By default it will use systrap mode, to switch it to kvm set k3s_gvisor_platform to kvm.<br>
If platform will be set to kvm, role will also load (and persist) corresponding module into kernel.<br>
It will also create RuntimeClass kubernetes object if you have variable k3s_gvisor_create_runtimeclass set to true (default).<br>
If you want to create it manually:
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
```
After that you should be able to launch gvisor-enabled pods by adding runtimeClassName to pod spec, ie
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gvisor-nginx
spec:
  runtimeClassName: gvisor
  containers:
    - name: nginx
      image: nginx
```
