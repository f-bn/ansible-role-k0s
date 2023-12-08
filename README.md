## Ansible role - K0s

### General informations

Ansible role to install and configure k0s Kubernetes distribution.

**Table of Contents**

- [General informations](#general-informations)
- [Role variables](#role-variables)
- [Examples](#examples)
- [Install and use this role](#install-and-use-this-role)

**Supported Platforms**

This role only supports Ubuntu Server LTS platforms:

  - Ubuntu 22.04 *Jammy Jellyfish*

**Supported k0s versions**

  - 1.28.x

**Requirements**

  - None

**References**

  - k0s : https://k0sproject.io/

**Implementation notes**

  - **Binary installation**: this roles installs k0s using the official binary, no other methods will be supported.
  - **Limited configuration by default**: this role embeds a basic configuration template with a limited set of feature for sake of simplicity. The role is designed to allow you to bring your own configuration templates (using the `k0s_controller_config_template` variable) instead of providing a generic template and falling in a template maintainability nightmare.
  - **Cluster deployments**: this role only installs the k0s binary, push various configurations and start the service on target hosts. You need to create a playbook to orchestrate a complete cluster bootstrap (see [examples](examples/bootstrap-cluster.yml) folder)

### Role variables

#### Global variables

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `k0s_version`                     | `1.28.4+k0s.0`               | Defines the version of the k0s to install                        |
| `k0s_role`                        | `controller`                 | Defines the cluster node role (valid values: `controller` or `worker`) | 
| `k0s_install_dir`                 | `/usr/local/bin`             | Defines the k0s binary installation directory                    |
| `k0s_data_dir`                    | `/var/lib/k0s`               | Defines the k0s data directory (state, etcd...)                  |
| `k0s_config_dir`                  | `/etc/k0s`                   | Defines the k0s configuration directory                          |
| `k0s_bin_url`                     | see [defaults](defaults/main.yml)| Defines the URL where to download the k0s binary             |
| `k0s_bin_update`                  | `false`                      | If set to `true`, force the k0s binary update                    |
| `k0s_join_token_path`             | `{{ k0s_config_dir }}/token` | Defines the absolute path to the node join token file            |
| `k0s_join_token`                  | `''`                         | Defines the token to use for configuring the node to join a cluster (valid for controller and worker join tokens) |

#### Controller variables

The following variables are only valid for **controller** nodes:

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `k0s_controller_config_path`      | `{{ k0s_config_dir }}/k0s.yaml`| Defines the absolute path to the rendered k0s configuration file |
| `k0s_controller_config_template`  | `k0s.default.yaml.j2`        | Defines a custom k0s configuration file to push from Ansible controller (can be a template) |
| `k0s_controller_components_disabled`| `[]`                       | Defines a list of k0s components to disable (more informations [here](https://docs.k0sproject.io/v1.27.2+k0s.0/configuration/?h=disabl#disabling-controller-components) |
| `k0s_controller_metrics_enabled`  | `false`                      | If set to `true`, k0s will provides a mechanism to expose controller system components metrics for observability (see [here](https://docs.k0sproject.io/v1.28.2+k0s.0/system-monitoring/)) |

#### Worker variables

The following variables are only valid for **worker** nodes:

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `k0s_worker_containerd_config_template`| `''`                    | If not empty, push a custom containerd configuration file from Ansible controller (can be a template) |
| `k0s_worker_profile`              | `''`                         | If not empty, defines the worker profile to apply to the worker node (must be defined in k0s configuration) |

### Examples

```YAML
# You can push a custom k0s configuration file/template from Ansible controller (controller)
k0s_controller_config_template: "/path/to/k0s.yaml.j2"

# Disable k0s components (controller)
k0s_controller_components_disabled:
  - helm
  - autopilot

# You can push a custom containerd configuration file/template from your Ansible controller (worker)
k0s_worker_containerd_config_template: '/path/to/containerd.toml'

# Custom kubelet worker profile
k0s_worker_profile: mycustomprofile
```

### Install and use this role

* Install the role using the command-line :

  ```shell
  $ ansible-galaxy role install git+https://github.com/f-bn/ansible-role-k0s.git
  ```

* You can also install the role in your projects using a `requirements.yml` file and `ansible-galaxy` command-line :

  ```YAML
  $ cat requirements.yml
  ---
  roles:
    - name: k0s
      src: https://github.com/f-bn/ansible-role-k0s.git
      scm: git
      version: '1.1.0'

  $ ansible-galaxy install-f -r requirements.yml
  ```

* Once the role is installed, you can use it in your playbooks :

  ```yaml
  - name: Deploy
    hosts: <hosts>
    roles:
      - role: k0s
  ```
