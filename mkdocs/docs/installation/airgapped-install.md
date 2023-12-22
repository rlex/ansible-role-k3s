# Airgapped installation
For environments without internet access, you can use
```yaml
k3s_install_mode: airgap
```
In that mode, role will download k3s binary and bootstrap images locally, and transfer them to target server from ansible runner.  
It will also work for gvisor.  
Please note that if you use [additional manifests installation](#adding-custom-kubernetes-manifests), you will need python3-kubernetes package installed on system - role assumes you have accessible OS distribution mirror configured on that airgapped node, otherwise installation will fail.  
If you can't get that package installed on your system, do not use automatic installation of manifests and set
```yaml
k3s_gvisor_create_runtimeclass: false
```
