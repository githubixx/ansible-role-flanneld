ansible-role-flanneld
=====================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). It installes flanneld which provides functionality for the Kubernetes pod network (makes it possible for pods on different hosts to communicate).

Requirements
------------

This role must be rolled out before Docker is installed. Additionally etcd must be running (but without that you won't have any part of Kubernetes running anyways ;-) ). During run the playbook will connect to the first node it finds in the `k8s_etcd` group and executes `etcdclt` there to add a new entry into etcd. That entry contains the flannel network config and it is located at "`flannel_etcd_prefix`/config".

Role Variables
--------------

```
k8s_interface: "tap0"
k8s_conf_dir: "/var/lib/kubernetes"
k8s_cni_conf_dir: "/etc/cni/net.d"

etcd_conf_dir: "/etc/etcd"
etcd_bin_dir: "/usr/local/bin"
etcd_client_port: 2379
etcd_certificates:
  - ca-etcd.pem
  - ca-etcd-key.pem
  - cert-etcd.pem
  - cert-etcd-key.pem

flannel_version: "v0.7.1"
flannel_etcd_prefix: "/kubernetes-cluster/network"
flannel_ip_range: "10.200.0.0/16"
flannel_cni_name: "podnet"
flannel_subnet_file_dir: "/run/flannel"
flannel_options_dir: "/etc/flannel"
flannel_bin_dir: "/usr/local/sbin"
flannel_ip_masq: "true"
flannel_cni_conf_file: "10-flannel"
```

Dependencies
------------

https://galaxy.ansible.com/githubixx/etcd/ installed.

Example Playbook
----------------

```
- hosts: k8s:children
  roles:
    - githubixx.kubernetes-flanneld
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)

