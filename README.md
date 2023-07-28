# K8S ansible role

This is a Ansible role to install K8S by using kubeadm (Tested on Ubuntu 22.04 / Debian 12)

## Requirements

Ansible 2.9

## Default variables

| Variable              | Value                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------- |
| KEEPALIVED_VROUTER_ID | 51                                                                                     |
| KEEPALIVED_IFACE      | enp1s0                                                                                 |
| CLUSTER_VIP           | 192.168.11.20                                                                          |
| CLUSTER_PORT          | 8443                                                                                   |
| CLUSTER_VERSION       | 1.26.6                                                                                 |
| CLUSTER_NAME          | k8s                                                                                    |
| CLUSTER_DOMAIN        | cluster.local                                                                          |
| KUBE_GPG_URL          | <https://packages.cloud.google.com/apt/doc/apt-key.gpg>                                |
| KUBE_GPG_PATH         | /etc/apt/keyrings/kubernetes-archive-keyring.gpg                                       |
| KUBE_POD_CIDR         | 10.244.0.0/16                                                                          |
| KUBE_SVC_CIDR         | 10.96.0.0/16                                                                           |
| KUBE_POD_NETWORK      | <https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml> |

## Example

```bash
### create the requirements.yml
cat <<< EOL > requirements.yml
- src: git+https://github.com/willyhutw/ansible-role-k8s.git
  name: k8s
EOL

### install role
ansible-galaxy install -r requirements.yml -p ./roles

### create the playbook.yml
cat <<< EOL > playbook.yml
- hosts: all
  become: yes
  gather_facts: yes
  roles:
    - roles/k8s
EOL

### create the inventory.yml
cat <<< EOL > inventory.yml
all:
  children:
    control_plane:
      children:
        control_plane_init:
          hosts:
            192.168.11.21:
              name: k8s-master-1
              priority: 120
        control_plane_join:
          hosts:
            192.168.11.22:
              name: k8s-master-2
              priority: 110
            192.168.11.23:
              name: k8s-master-3
              priority: 100
    node:
      hosts:
        192.168.11.31:
        192.168.11.32:
        192.168.11.33:
  vars:
    ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
EOL

### run it!
ansible-playbook -i inventory.yml playbook.yml \
-e ansible_ssh_user=willyhu \
-e KEEPALIVED_VIP=192.168.11.20 \
-e CLUSTER_VERSION=1.26.7 \
-e CLUSTER_NAME=mycluster \
-e CLUSTER_DOMAIN=mydomain
```
