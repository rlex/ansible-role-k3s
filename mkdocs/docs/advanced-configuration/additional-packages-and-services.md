# Additional packages and services
Sometimes certain software requires certain packages installed on host system. Some of examples are distributed filesystems like longhorn and openebs which require iscsid.  
While it's better to manage such software with dedicated roles, i included that variables for simplicity. If you want openebs-jiva or longhorn to work, you can specify
```yaml
k3s_additional_packages:
  - open-iscsi
k3s_additional_services:
  - iscsid
```
open-iscsi will be installed and iscsid service will be both started and enabled at boot-time before k3s installation
