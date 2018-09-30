Changelog
---------

**v6.0.2_r0.10.0**

- update README

**v6.0.1_r0.10.0**

- fix order of commands in flanneld.service.j2 template

**v6.0.0_r0.10.0**

- variable `flannel_cni_name` no longer needed
- template `cni-flannel.conf.j2` no longer needed (see `flannel_cni_conf` variable)
- new variables `flannel_systemd_execstartpre` and `flannel_systemd_execstartpost`
- `etcd_conf_dir` variable no longer needed
- introduce `flannel_cni_interface` variable

**v5.0.0_r0.10.0**

- working on Ubuntu 18.04
- PeerVPN is no requirement, could be any VPN solution that supports fully meshed VPN's like WireGuard
- fix etcdctl endpoints
- gather facts before run other tasks

**v4.0.0_r0.10.0**

- upgrade to Flannel v0.10.0
- major refactoring
- introduce flexible parameter settings for flannel daemon via `flannel_settings` and `flannel_settings_user`

**>= v3.0.0_r0.9.1**

- no change log available (see commit history if needed)
