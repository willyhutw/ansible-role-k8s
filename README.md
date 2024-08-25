# K8S ansible role

This is a ansible role to deploy kubernetes cluster by using kubeadm (Tested on Ubuntu 22.04 and Debian 12).

## Requirements

Ansible 2.10.x or above

## Default variables

| Variable              | Value                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------- |
| SINGLE_MASTER_NODE    | false                                                                                  |
| KEEPALIVED_VROUTER_ID | 51                                                                                     |
| KEEPALIVED_IFACE      | enp1s0                                                                                 |
| CLUSTER_VIP           | 192.168.12.20                                                                          |
| CLUSTER_PORT          | 8443                                                                                   |
| CLUSTER_NAME          | k8s                                                                                    |
| CLUSTER_DOMAIN        | cluster.local                                                                          |
| CLUSTER_BRANCH        | v1.30                                                                                  |
| CLUSTER_VERSION       | 1.30.4                                                                                 |
| KUBE_GPG_URL_PREFIX   | <https://pkgs.k8s.io/core:/stable:>                                                    |
| KUBE_GPG_PATH         | /etc/apt/keyrings/kubernetes-apt-keyring.gpg                                           |
| KUBE_POD_CIDR         | 10.244.0.0/16                                                                          |
| KUBE_SVC_CIDR         | 10.96.0.0/16                                                                           |
| KUBE_POD_NETWORK      | <https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml> |

## Example

### DO NOT NEED to clone this repo, we will install this role by running ansible-galaxy command.

```bash
### create the requirements.yml
cat << EOL > requirements.yml
- src: git+https://github.com/willyhutw/ansible-role-k8s.git
  name: k8s
EOL

### install this role
ansible-galaxy install -r requirements.yml -p ./roles

### create the playbook.yml
cat << EOL > playbook.yml
- hosts: all
  become: yes
  gather_facts: yes
  roles:
    - roles/k8s
EOL

### --- deploy 3-node control plane ---
cat << EOL > inventory_multi_master.yml
all:
  children:
    control_plane:
      children:
        control_plane_init:
          hosts:
            192.168.12.21:
              name: k8s-master-1
              priority: 120
        control_plane_join:
          hosts:
            192.168.12.22:
              name: k8s-master-2
              priority: 110
            192.168.12.23:
              name: k8s-master-3
              priority: 100
    node:
      hosts:
        192.168.12.31:
        192.168.12.32:
        192.168.12.33:
  vars:
    ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
EOL

### pass variables to rewrite the defaults if you need.
ansible-playbook -i inventory_multi_master.yml playbook.yml \
-e ansible_ssh_user=$TARGET_MACHINE_USERNAME \
-e KEEPALIVED_VIP=192.168.12.20

### --- deploy single-node control plane ---
cat << EOL > inventory_single_master.yml
all:
  children:
    control_plane:
      children:
        control_plane_init:
          hosts:
            192.168.12.21:
              name: k8s-master-1
    node:
      hosts:
        192.168.12.31:
        192.168.12.32:
        192.168.12.33:
  vars:
    ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
EOL

### remember to rewrite the default SINGLE_MASTER_NODE variable.
ansible-playbook -i inventory_single_master.yml playbook.yml \
-e ansible_ssh_user=$TARGET_MACHINE_USERNAME \
-e KEEPALIVED_VIP=192.168.12.20 \
-e SINGLE_MASTER_NODE=true
```
