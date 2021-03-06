# k8s-on-openstack

An opinionated way to deploy a Kubernetes cluster on top of an OpenStack cloud. It uses [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) and [ansible](https://www.ansible.com/) to deploy a k8s cluster with [Calico CNI](https://github.com/projectcalico/cni-plugin) as networking substrate.

It has been verified to work with
- Ubuntu 18.04 LTS (update [group_vars/all.yaml](group_vars/all.yaml)) 
- Ubuntu 20.04 LTS (no changes required)

## Getting started

### Environment variables

The following **mandatory** environment variables need to be set before calling `ansible-playbook`:

* `OS_*`: standard OpenStack environment variables such as `OS_AUTH_URL`, `OS_USERNAME`, ...
* `KEY`: name of an existing SSH keypair

The following **optional** environment variables can also be set:

* `NAME`: name of the Kubernetes cluster, used to derive instance names, `kubectl` configuration and security group name
* `IMAGE`: name of an existing Ubuntu 20.04 image
* `EXTERNAL_NETWORK`: name of the neutron external network, defaults to 'public'
* `FLOATING_IP_POOL`: name of the floating IP pool
* `FLOATING_IP_NETWORK_UUID`: uuid of the floating IP network (required for LBaaSv2)
* `USE_OCTAVIA`: try to use Octavia instead of Neutron LBaaS, defaults to False
* `USE_LOADBALANCER`: assume a loadbalancer is used and allow traffic to nodes (default: false)
* `SUBNET_CIDR` the subnet CIDR for OpenStack's network (default: `10.8.10.0/24`)
* `POD_SUBNET_CIDR` CIDR of the POD network (default: `10.96.0.0/16`)
* `CLUSTER_DNS_IP`: IP address of the cluster DNS service passed to kubelet (default: `10.96.0.10`)
* `BLOCK_STORAGE_VERSION`: version of the block storage (Cinder) service, defaults to 'v2'
* `IGNORE_VOLUME_AZ`: whether to ignore the AZ field of volumes, needed on some clouds where AZs confuse the driver, defaults to False.
* `NODE_MEMORY`: how many MB of memory should nodes have, defaults to 4GB
* `NODE_FLAVOR`: allows to configure the exact OpenStack flavor name or ID to use for the nodes. When set, the `NODE_MEMORY` setting is ignored.
* `NODE_COUNT`: how many nodes should we provision, defaults to 3
* `NODE_AUTO_IP` assign a floating IP to nodes, defaults to False
* `NODE_DELETE_FIP`: delete floating IP when node is destroyed, defaults to True
* `NODE_BOOT_FROM_VOLUME`: boot node instances using boot from volume. Useful on clouds with only boot from volume
* `NODE_TERMINATE_VOLUME`: delete the root volume when each node instance is destroy, defaults to True
* `NODE_VOLUME_SIZE`: size of each node volume. defaults to 64GB
* `NODE_EXTRA_VOLUME`: create an extra unmounted data volume for each node, defaults to False
* `NODE_EXTRA_VOLUME_SIZE`: size of extra data volume for each node, defaults to 80GB
* `NODE_DELETE_EXTRA_VOLUME`: delete the extra data volume for each node when node is destroy, defaults to True
* `MASTER_BOOT_FROM_VOLUME`: boot the master instance on a volume for data persistence, defaults to True
* `MASTER_TERMINATE_VOLUME`: delete the volume when master instance is destroy, defaults to True
* `MASTER_VOLUME_SIZE`: size of the master volume. default to 64GB
* `MASTER_MEMORY`: how many MB of memory should master have, defaults to 4 GB
* `MASTER_FLAVOR`: allows to configure the exact OpenStack flavor name or ID to use for the master. When set, the `MASTER_MEMORY` setting is ignored.
* `AVAILABILITY_ZONE`: the availability zone to use for nodes and the default `StorageClass` (defaults to `nova`). This affects `PersistentVolumeClaims` without explicit a storage class.
* `HELM_REPOS`: a list of additional helm repos to add, separated by semicolons. Example: `charts* https://github.com/helm/charts;mycharts https://github.com/dev/mycharts`
* `HELM_INSTALL`: a list of helm charts and their parameters to install, separated by semicolons. Example: `mycharts/mychart;charts/somechart --name somechart --namespace somenamespace`

### Spin up a new cluster

```console
$ ansible-playbook site.yaml
```

### Destroy the cluster

```console
$ ansible-playbook destroy.yaml
```

### Upgrade the cluster (requires update; non-functional)

The `upgrade.yaml` playbook implements the upgrade steps described in https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-11/
After editing in `group_vars/all.yaml` the `kubernetes_version` and `kubernetes_ubuntu_version` variables, you can run the following commands.

```console
$ ansible-playbook upgrade.yaml
$ ansible-playbook site.yaml
```

## Prerequisites

* Ansible (tested with version 2.9.1)
* Shade library required by Ansible OpenStack modules (`python-shade` for Debian)

## CI/CD

The following environment variables needs to be defined:

* `OS_AUTH_URL`
* `OS_PASSWORD`
* `OS_USERNAME`
* `OS_DOMAIN_NAME`

# Authors

* François Deppierraz <francois.deppierraz@infraly.ch>
* Oli Schacher <oli.schacher@switch.ch>
* Saverio Proto <saverio.proto@switch.ch>
* @HaseHarald <https://github.com/HaseHarald>
* Dennis Pfisterer <https://github.com/pfisterer>

# References

* https://kubernetes.io/docs/getting-started-guides/kubeadm/
* https://github.com/kubernetes/dashboard#kubernetes-dashboard
