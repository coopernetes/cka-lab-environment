# Kubernetes lab for CKA study
This is a simple set of configurations, Ansible tasks & playbooks to deploy & manage some Debian virtual machines for the purposes of studying for the [CKA](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) exam.

Targeting the current [CKA exam version of Kubernetes, v1.24.4](https://github.com/cncf/curriculum).

## Topology
So far, just six virtual machines running on Virtualbox. Connected via one NAT NIC (for Internet access to download dependencies) and one host-only internal NIC for communicating with my WSL Ansible control host.

These playbooks follow the [options for software load balancing](https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing) to also deploy keepalived and haproxy to load balance across the 3 master nodes.

## Usage
It's assumed that you have a valid Ansible inventory specified in `/etc/ansible/hosts` with two groups, one for the masters and another for worker nodes. There is also a provided [hosts](hosts) file that can be specified with the `-i` option.

```
[kubernetes]
debian0[1:6].lab

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
$ ansible-playbook -K -i hosts site.yml
```

## Upgrades
The following tags can be used to upgrade specific components and remove dpkg hold. 

**Note:** This playbook does not drain any nodes of active pods. 

To upgrade a cluster:

1. Change `kubernetes_version` in [group_vars/all](group_vars/all) or set it via extra-args.

2. Run kubeadm & kubectl upgrades on a master node. Use `--limit <nodename>` to choose which one to perform the upgrade on.
    ```shell
    ansible-playbook -i hosts site.yml --tags "upgrade,kubeadm,kubectl" --limit debian01.lab
    ```

3. Upgrade the kubectl on the master nodes. 
    ```shell
    ansible-playbook -i hosts site.yml --tags "upgrade,kubelet" --limit "masters"
    ```

4. I forget what to do at this step :(

5. Upgrade the worker nodes. 
    ```shell
    ansible-playbook -i hosts site.yml --tags "upgrade,kubeadm,kubectl,kubelet" --limit "nodes"
    ```

## Rest
Use `kubeadm reset` via ansible ad-hoc to completely reset a cluster node.

```shell
ansible kubernetes --become -K -i hosts -a 'kubeadm reset -f --ignore-preflight-errors all'
```

