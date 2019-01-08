ansible-role-flanneld
=====================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-worker/). It installes flanneld which provides functionality for the Kubernetes pod network (makes it possible for pods on different hosts to communicate).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `7.0.0+0.10.0` means this is version `7.0.0` of this role and it's meant to be used with Flannel version `0.10.0` (but maybe also works with newer versions). If the role itself changes `X.Y.Z` before `+` will increase. If the Flannel version changes `X.Y.Z` after `+` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Flannel release.

Requirements
------------

This role must be rolled out before Docker is installed (see https://github.com/githubixx/ansible-role-docker). Additionally etcd (see https://github.com/githubixx/ansible-role-etcd) must be running (but without that you won't have any part of Kubernetes running anyways ;-) ). During run the playbook will connect to the first node it finds in the `k8s_etcd` group and executes `etcdclt` there to add a new entry into etcd. That entry contains the flannel network config and it is located at "`flannel_etcd_prefix`/config".

Changelog
---------

see [CHANGELOG.md](https://github.com/githubixx/ansible-role-flanneld/blob/master/CHANGELOG.md)

Role Variables
--------------

```
# The interface on which the K8s services should listen on. As all cluster
# communication should use the VPN interface the interface name is
# normally "wg0", "tap0" or "peervpn0".
k8s_interface: "tap0"
# Directory where the K8s certificates and other configuration are stored
# on the Kubernetes hosts.
k8s_conf_dir: "/var/lib/kubernetes"
# CNI network plugin directory
k8s_cni_conf_dir: "/etc/cni/net.d"
# The directory from where to copy the K8s certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"

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
flannel_cni_interface: "cni0"
flannel_subnet_file_dir: "/run/flannel"
flannel_options_dir: "/etc/flannel"
flannel_bin_dir: "/usr/local/sbin"
flannel_cni_conf_file: "10-flannel"

flannel_systemd_restartsec: "5"
flannel_systemd_limitnofile: "40000"
flannel_systemd_limitnproc: "1048576"
# "ExecStartPre" directive in flannel's systemd service file. This command
# is executed before flannel service starts.
flannel_systemd_execstartpre: "/bin/mkdir -p {{flannel_subnet_file_dir}}"
# "ExecStartPost" directive in flannel's systemd service file. This command
# is execute after flannel service is started. If you run in Hetzner cloud
# this may be important. In this case it changes the TX checksumming offload
# parameter for the "flannel.1" interface. It seems that there is a 
# (kernel/driver) checksum offload bug with flannel vxlan encapsulation 
# (all inside UDP) inside WireGuard encapsulation.
# flannel_systemd_execstartpost: "/sbin/ethtool -K flannel.1 tx off"

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

flannel_cni_conf: |
  {
    "name": "{{flannel_cni_interface}}",
    "plugins": [
      {
        "type": "flannel",
        "delegate": {
          "hairpinMode": true,
          "isDefaultGateway": true
        }
      },
      {
        "type": "portmap",
        "capabilities": {
          "portMappings": true
        }
      }
    ]
  }
```

The settings for Flannel daemon defined in `flannel_settings` can be overriden by defining a variable called `flannel_settings_user`. You can also add additional settings by using this variable. E.g. to override `healthz-ip` default value and add `kubeconfig-file` setting add the following settings to `group_vars/all.yml` or where ever it fit's best for you:

```
flannel_settings_user:
  "healthz-ip": "1.2.3.4"
  "kubeconfig-file": "/etc/k8s/k8s.cfg"
```

A word about `flannel_systemd_execstartpost: "/sbin/ethtool -K flannel.1 tx off"` which is commented out by default. If Pod-to-Pod communication works for you (across hosts) but Pod-to-Service or Node-to-Service doesn't work (also across hosts) then setting this variable and the content might help. At least it helped running Flannel traffic via WireGuard VPN at Hetzner cloud. It seems that TX offloading is not working in this case and this causes checksum errors. You can identify this problem like this. Log into two worker nodes via SSH. Identify a service that is e.g. running on `worker01` and try to access it on `worker02`. E.g. the `kube-dns` or `coredns` pod is a good candidate as it runs basically on every Kubernetes cluster. Desipe the fact that DNS queries normally send via UDP protocol DNS also works with TCP. So assume `kube-dns` is running on `worker01` and on `worker02` you try to connect to the DNS service via

```
telnet 10.32.0.254 53
Trying 10.32.0.254...
```

But as you can see nothing happens. So let's run `tcpdump` on `worker01` e.g. `tcpdump -nnvvvS -i any src host <ip_of_worker02>`. Now we capture every traffic from `worker02` on `worker01`. Execute `telnet 10.32.0.254 53` again on `worker02` and you may see something line this:

```
22:17:18.500515 IP (tos 0x0, ttl 64, id 32604, offset 0, flags [none], proto UDP (17), length 110)
    10.8.0.112.43977 > 10.8.0.111.8472: [udp sum ok] OTV, flags [I] (0x08), overlay 0, instance 1
IP (tos 0x10, ttl 63, id 10853, offset 0, flags [DF], proto TCP (6), length 60)
    10.200.94.0.40912 > 10.200.1.37.53: Flags [S], cksum 0x74e3 (incorrect -> 0x890f), seq 3891002740, win 43690, options [mss 65495,sackOK,TS val 2436992709 ecr 0,nop,wscale 7], length 0
...
```

If you have a closer look in the last line you see in this example `cksum 0x74e3 (incorrect -> 0x890f)` and that's the problem. The `SYN` packet is dropped because the checksum is wrong. Now if you disable checksum offloading for `flannel.1` interface on all hosts (again at least on Hetzner cloud) it works:

```
telnet 10.32.0.254 53
Trying 10.32.0.254...
Connected to 10.32.0.254.
Escape character is '^]'.
...
```

And if we now have a look at the `tcpdump` output again

```
22:34:26.605677 IP (tos 0x0, ttl 64, id 794, offset 0, flags [none], proto UDP (17), length 110)
    10.8.0.112.59225 > 10.8.0.111.8472: [udp sum ok] OTV, flags [I] (0x08), overlay 0, instance 1
IP (tos 0x10, ttl 63, id 16989, offset 0, flags [DF], proto TCP (6), length 60)
    10.200.94.0.42152 > 10.200.1.37.53: Flags [S], cksum 0xd49b (correct), seq 1673757006, win 43690, options [mss 65495,sackOK,TS val 2438020768 ecr 0,nop,wscale 7], length 0
...
```

you'll see in the last line the checksum is correct. You can inspect the state of protocol offload and other features of the Flannel interface with `ethtool -k flannel.1`.

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

