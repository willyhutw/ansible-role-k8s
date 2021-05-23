# K8S ansible role

This is a Ansible role to install K8S by using kubeadm (Tested on Ubuntu 20.04)

## Requirements

Ansible 2.9

## Default variables

|Variable|Value|Description|
|---|---|---|
```DOCKER_GPG_URL```|https://download.docker.com/linux/ubuntu/gpg|
```DOCKER_GPG_PATH```|/usr/share/keyrings/docker-archive-keyring.gpg|
```DOCKER_VERSION```|`5:20.10.6~3-0~ubuntu-focal`|
```KUBE_GPG_URL```|https://packages.cloud.google.com/apt/doc/apt-key.gpg|
```KUBE_GPG_PATH```|/usr/share/keyrings/kubernetes-archive-keyring.gpg|
```KUBE_VERSION```|1.21.1-00|
```KUBE_POD_NETWORK```|https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml|
```KUBE_POD_CIDR```|10.244.0.0/16|
```KUBE_JOIN_CMD_PATH```|/vagrant_data/join.cmd|

## Example
```
### create the requirements.yml
- src: git+https://github.com/willyhutw/ansible-role-k8s.git
  name: k8s

### install role
ansible-galaxy install -r requirements.yml -p ./roles

### create the playbook.yml
- hosts: all
  become: yes
  gather_facts: yes
  roles:
    - roles/k8s

### create the inventory.yml 
all:
  children:
    master:
      hosts:
        192.168.33.11:
    worker:
      hosts:
        192.168.33.21:
        192.168.33.22:
  vars:
    ansible_ssh_common_args: "-o StrictHostKeyChecking=no"

### run it!
ansible-playbook -i inventory.yml playbook.yml \
-e ansible_ssh_user=vagrant \
-e ansible_ssh_pass=vagrant
```

