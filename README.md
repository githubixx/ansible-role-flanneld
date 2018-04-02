ansible-role-flanneld
=====================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). It installes flanneld which provides functionality for the Kubernetes pod network (makes it possible for pods on different hosts to communicate).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org) (well kind of). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `v4.0.0_r0.10.0` means this is version `4.0.0` of this role and it's meant to be used with Flannel version `0.10.0` (but maybe also works with higher versions). If the role itself changes `vX.Y.Z` will increase. If the Flannel version changes `rX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Flannel release.

Requirements
------------

This role must be rolled out before Docker is installed. Additionally etcd must be running (but without that you won't have any part of Kubernetes running anyways ;-) ). During run the playbook will connect to the first node it finds in the `k8s_etcd` group and executes `etcdclt` there to add a new entry into etcd. That entry contains the flannel network config and it is located at "`flannel_etcd_prefix`/config".

Changelog
---------

**v4.0.0_r0.10.0**

- upgrade to Flannel v0.10.0
- major refactoring
- introduce flexible parameter settings for flannel daemon via `flannel_settings` and `flannel_settings_user`

**>= v3.0.0_r0.9.1**

- no change log available

Role Variables
--------------

```
# The interface on which the K8s services should listen on. As all cluster
# communication should use the PeerVPN interface the interface name is
# normally "tap0" or "peervpn0".
k8s_interface: "tap0"
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# CNI network plugin settings
k8s_cni_conf_dir: "/etc/cni/net.d"
# The directory from where to copy the K8s certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"

etcd_conf_dir: "/etc/etcd"
etcd_bin_dir: "/usr/local/bin"
etcd_client_port: 2379
etcd_certificates:
  - ca-etcd.pem
  - ca-etcd-key.pem
  - cert-etcd.pem
  - cert-etcd-key.pem

flannel_version: "v0.10.0"
flannel_etcd_prefix: "/kubernetes-cluster/network"
flannel_ip_range: "10.200.0.0/16"
flannel_backend_type: "vxlan"
flannel_cni_name: "podnet"
flannel_subnet_file_dir: "/run/flannel"
flannel_options_dir: "/etc/flannel"
flannel_bin_dir: "/usr/local/sbin"
flannel_cni_conf_file: "10-flannel"

flannel_systemd_restartsec: "5"
flannel_systemd_limitnofile: "40000"
flannel_systemd_limitnproc: "1048576"

flannel_settings:
  "etcd-cafile": "{{k8s_conf_dir}}/ca-etcd.pem"
  "etcd-certfile": "{{k8s_conf_dir}}/cert-etcd.pem"
  "etcd-keyfile": "{{k8s_conf_dir}}/cert-etcd-key.pem"
  "etcd-prefix": "{{flannel_etcd_prefix}}"
  "iface": "{{k8s_interface}}"
  "public-ip": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "subnet-file": "{{flannel_subnet_file_dir}}/subnet.env"
  "ip-masq": "true"
  "healthz-ip": "0.0.0.0"
  "healthz-port": "0" # 0 = disable
```

The settings for Flannel daemon defined in `flannel_settings` can be overriden by defining a variable called `flannel_settings_user`. You can also add additional settings by using this variable. E.g. to override `healthz-ip` default value and add `kubeconfig-file` setting add the following settings to `group_vars/all.yml` or `group_vars/k8s.yml` or where ever it fit's best for you:

```
flannel_settings_user:
  "healthz-ip": "1.2.3.4"
  "kubeconfig-file": "/etc/k8s/k8s.cfg"
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

