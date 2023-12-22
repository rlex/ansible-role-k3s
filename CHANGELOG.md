#### Master

#### 2.0.0
Version 2.0.0 deprecates and renames following variables to follow single style: 

* k3s_agent_additional_config -> k3s_agent_extra_config
* k3s_master_additional_config -> k3s_extra_services
* k3s_kubelet_additional_config -> k3s_extra_config_files
* k3s_additional_packages -> k3s_extra_manifests
* k3s_additional_services -> k3s_agent_extra_config
* k3s_additional_config_files -> k3s_agent_extra_config
* k3s_additional_manifests -> k3s_kubelet_extra_config
* k3s_node_extra_args -> k3s_agent_extra_args
* k3s_node_external_ip -> k3s_external_ip
* k3s_node_ip -> k3s_internal_ip
* k3s_server_flannel_backend -> k3s_flannel_backend
* k3s_server_disable -> k3s_master_disable
* k3s_server_extra_args -> k3s_master_extra_args
* k3s_node -> k3s_agent

They are still usable and symlinked to new variables, but they will be removed in future versions

New features and bugfixes:
* Migrate documentation to mkdocs to avoid gigantic readme
* Sets new default versions for k3s and gvisor
* Allow deploying additional k8s manifests from ansible role
* Supports crun runtime installation from role
* Do not init etcd if only single master is present
* Automated testing now verifies that pods are starting (previously only checked for k3s service start)
* Automatically detects CPU architecture for k3s, gvisor and crun
* Automatically loads and persists kvm module if gvisor platform is set to kvm
* Supports airgapped installation mode
* Allows specifying custom path for containerd template
* Brings role to modern standards, using FQCNs for ansible modules and removing deprecated parameters from said modules
* Fixes all yaml and ansible linter warnings

#### 1.0.0
Initial release
