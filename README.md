# Kubernetes lab for CKA study
This is a simple set of configurations, Ansible tasks & playbooks to deploy & manage some Debian virtual machines for the purposes of studying for the [CKA](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) exam.

Targeting the current [CKA exam version of Kubernetes, v1.24.4](https://github.com/cncf/curriculum).

## Topology
32GB + Intel Xeon E1231v3 CPU on the host machine.

Six (6) virtual machines act as the cluster.
* Three (3) Ubuntu 22.04 VMs acting as cluster master, API server and etcd. 3GB/2CPU
  - kubeadm, kubelet and kubectl
  - keepalived + haproxy for high availability. See [options for software load balancing](https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing) for details. 
* Three (3) Ubuntu 22.04 VMs acting as worker nodes. 2GB/2CPU
  - kubelet

Each server has two NICs, one host-only private network (192.168.56.0/24) and the other bridged to the host for Internet access (192.168.1.0/24).

An additional VM is used as an Ansible control host and development.

## Usage
It's assumed that you have a valid Ansible inventory specified in `/etc/ansible/hosts` with two groups, one for the masters and another for worker nodes. There is also a provided [hosts](hosts) file that can be specified with the `-i` option.

```
[kubernetes]
ubuntu0[1:6].lab

[masters]
ubuntu0[1:3].lab

[nodes]
ubuntu0[4:6].lab
```

I'm also using the [common role](roles/common/tasks/main.yaml) to configure the `/etc/hosts` file for internal name resolution. 

Playbook follows the kubeadm installation instructions found here: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
$ ansible-playbook -i inventory/hosts site.yaml
```

## Upgrades
The following tags can be used to upgrade specific components and remove dpkg hold. 

**Note:** This playbook does not drain any nodes of active pods. 

To upgrade a cluster:

1. Change `kubernetes_version` in [group_vars/all](group_vars/all) or set it via extra-args.

2. Run kubeadm & kubectl upgrades on a master node. Use `--limit <nodename>` to choose which one to perform the upgrade on.
    ```shell
    ansible-playbook -i hosts site.yaml --tags "upgrade,kubeadm,kubectl" --limit debian01.lab
    ```

3. Upgrade the kubectl on the master nodes. 
    ```shell
    ansible-playbook -i hosts site.yaml --tags "upgrade,kubelet" --limit "masters"
    ```

4. Drain the worker nodes.
   ```shell
   ansible nodes -i hosts -K --become -a 'kubectl drain $(hostname)'
   ```

5. Upgrade the worker nodes. 
    ```shell
    ansible-playbook -i hosts site.yaml --tags "upgrade,kubeadm,kubectl,kubelet" --limit "nodes"
    ``

6. Uncordon the nodes.
   ```shell
   ansible nodes -i hosts -K --become -a 'kubectl uncordon $(hostname)'
   ```

## Rest
Use `kubeadm reset` via ansible ad-hoc to completely reset a cluster node.

```shell
ansible kubernetes --become -K -i hosts -a 'kubeadm reset -f --ignore-preflight-errors all'
```

