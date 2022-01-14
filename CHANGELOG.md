Changelog
---------

**8.0.0+0.16.1

- update to flannel v0.16.1
- remove unneeded directories
- make ansible-lint happy
- remove Ubuntu 16.04 support
- add role_name/namespace to meta/main.yml

**no version update**

- needed to fix (rename) tag names to adhere sematic versioning scheme (SemVer). The following tags are affected:

```
v1.0.0_r0.7.1   -> 1.0.0+0.7.1
v1.0.1_r0.7.1   -> 1.0.1+0.7.1
v2.0.0_r0.9.0   -> 2.0.0+0.9.0
v2.0.1_r0.9.1   -> 2.0.1+0.9.1
v2.0.2_r0.9.1   -> 2.0.2+0.9.1
v3.0.0_r0.9.1   -> 3.0.0+0.9.1 
v4.0.0_r0.10.0  -> 4.0.0+0.10.0
v5.0.0_r0.10.0  -> 5.0.0+0.10.0
v6.0.0_r0.10.0  -> 6.0.0+0.10.0
v6.0.1_r0.10.0  -> 6.0.1+0.10.0
v6.0.2_r0.10.0  -> 6.0.2+0.10.0
```

`7.0.2+0.10.0` and `7.0.1+0.10.0` are not affected.

**7.0.2+0.10.0**

- Fix for Kubernetes v1.16.x. Introducing `flannel_cni_spec_version` variable. Kubernetes v1.16 requires `cniVersion` specified in `10-flannel.conflist` which is the latest supported CNI specification.

**7.0.1+0.10.0**

- Fixed a spacing issue: https://github.com/githubixx/ansible-role-flanneld/pull/8

**7.0.0+0.10.0**

- use correct semantic versioning as described in https://semver.org. Needed for Ansible Galaxy importer as it now insists on using semantic versioning.
- make Ansible linter happy
- no major changes but decided to start a new major release as versioning scheme changed quite heavily

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
