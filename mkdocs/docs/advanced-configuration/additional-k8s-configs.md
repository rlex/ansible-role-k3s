# Creating additional kubernetes configs
Sometimes you need to create additional config files for adding to kubernetes installation.  
For example, you want to trace kubelet, which requires separate config file for tracing configuration.  
Variable ```k3s_extra_config_files``` will take care of that.  
All additional config files will go to ```/etc/rancher/k3s``` directory, with name specified in name block and with content specified in content.  
This action will happen on pre-configuration stage, before k3s installation.  

Example:
```yaml
k3s_extra_config_files:
  - name: apiserver-tracing.yaml
    content: |
      apiVersion: apiserver.config.k8s.io/v1alpha1
      kind: TracingConfiguration
      endpoint: 127.0.0.1:4317
      samplingRatePerMillion: 100
```

Will result in file ```/etc/rancher/k3s/apiserver-tracing.yaml``` with following content:
```yaml
apiVersion: apiserver.config.k8s.io/v1alpha1
kind: TracingConfiguration
endpoint: 127.0.0.1:4317
samplingRatePerMillion: 100
```

Please note that no additional formatting or processing is happening on that stage, so you need to take care about all indentation and other formatting things.  
Additionally, editing any of those files will trigger k3s restart
