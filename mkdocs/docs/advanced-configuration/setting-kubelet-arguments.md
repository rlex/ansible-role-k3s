# Setting kubelet arguments
To pass arguments for kubelet, you can use ```k3s_kubelet_additional_config``` variable:
```yaml
k3s_kubelet_additional_config:
  - "image-gc-high-threshold=40"
  - "image-gc-low-threshold=30"
```
