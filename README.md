# Kubernetes lab for CKA study
This is a simple set of configurations, Ansible tasks & playbooks to deploy & manage some Debian virtual machines for the purposes of studying for the [CKA](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) exam.

Targeting the current [CKA exam version of Kubernetes, v1.21.4](https://github.com/cncf/curriculum).

## Topology
So far, just six virtual machines running on Virtualbox. Connected via one NAT NIC (for Internet access to download dependencies) and one host-only internal NIC for communicating with my WSL Ansible control host.

## Usage
It's assumed that you have a valid Ansible inventory specified in `/etc/ansible/hosts` with two groups, one for the masters and another for worker nodes.

```
[masters]
debian01.lab
debian02.lab
debian03.lab

[nodes]
debian04.lab
debian05.lab
debian06.lab
```

I'm also using the [common role](roles/common/tasks/main.yml) to configure the `/etc/hosts` file for internal name resolution. 

Playbook follows the kubeadm installation instructions found here: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
$ ansible-playbook -K site.yml
```
