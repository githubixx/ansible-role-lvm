<!--
Copyright (C) 2021-2022 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

ansible-role-lvm
================

This Ansible roles installs Linux Logical Volume Manager (LVM) resources like `Volume Groups` (VG), `Logical Volumes` (LV) and handles `filesystems` creation and `mountpoints`.

Changelog
---------

see [CHANGELOG](https://github.com/githubixx/ansible-role-lvm/blob/master/CHANGELOG.md)

Role Variables
--------------

By default no LVM resources are created. See examples below. The `lvm_vgs` key is the entrypoint for all LVM resources that should be created.

```yaml
lvm_vgs: []
```

First a few examples about creating LVM resources (how deletion is handled see further down below).

Create a volume group (VG) called `test-vg-01` which uses device `/dev/vdb` as physical volume:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
```

All three parameters are required! [vgname](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-vg) is the name of the Volume Group. [pvs](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-pvs) is a list of comma-separated devices to use as physical devices in this volume group. [state](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-state) controls if the volume group exists.  
To delete a VG you need to specify `state: absent` and also `force: true`. In contrast to the `force` parameter of the [community.general.lvg](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html) module `force: true` won't delete VGs that still contains (a) logical volume(s)! This is to make it harder to delete data by accident.

Create a volume group (VG) called `test-vg-01` which uses devices `/dev/vdb` and `/dev/vdc` as physical volumes:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb,/dev/vdc
    state: present
```

Currently the following additional [community.general.lvg](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html) parameters are supported:

- [vg_options](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-vg_options)
- [pesize](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-pesize)
- [pv_options](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html#parameter-pv_options)

As previous example but also create a logical volume `test-lv-01` and allocate `10%` of the available VG space:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
    lvm_lvs:
      - lvname: test-lv-01
        size: 10%VG
        state: present
```

All logical volumes a volume group contains are listed under `lvm_lvs` key. `lvname`, `size` and `state` parameters are required! [lvname](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-lv) is the name of the logical volume. [size](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-size) specifies the size of the logical volume, according to `lvcreate(8) --size`, by default in megabytes or optionally with one of [bBsSkKmMgGtTpPeE] units; or according to `lvcreate(8) --extents` as a percentage of [VG|PVS|FREE]; Float values must begin with a digit. [state](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-state) controls if the logical volume exists. To delete a LV you need to specify `state: absent` and also `force: true`.

Currently also the following additional [community.general.lvol](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameters) parameters are supported:

- [active](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-active)
- [opts](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-opts)
- [pvs](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-pvs)
- [thinpool](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html#parameter-thinpool)

As previous example but also create a `ext4` filesystem using logical volume `test-lv-01` as device for the filesystem:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
    lvm_lvs:
      - lvname: test-lv-01
        size: 10%VG
        state: present
        fs:
          type: ext4
          state: present
```

The `fs` key specifies that the logical volume should contain a filesystem. [type](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html#parameter-fstype) specifies the filesystem that should be used. `state: present` specifies that the filesystem is created if it doesn't already exist. In case of `state: absent` filesystem signatures on the device are wiped if it contains a filesystem (as known by `blkid`). When `state: absent`, all other options but the device are ignored, and the module doesn't fail if the device doesn't actually exist. In case of `state: absent` also `force: true` is needed otherwise the filesystem won't be touched. In contrast to the [force](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html#parameter-force) parameter of the [community.general.filesystem](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html) module `force: true` won't override an existing filesystem. You really need to wipe the old one first to be able to create an new filesystem! Again this is to avoid deletion by accident.

As previous example but also create a directory `/mnt1` and mount the `ext4` filesystem there. This also creates an entry in `/etc/fstab` (by default).

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
    lvm_lvs:
      - lvname: test-lv-01
        size: 10%VG
        state: present
        fs:
          type: ext4
          state: present
          mountpoint:
            state: mounted
            path:
              name: /mnt1
```

The `mountpoint` key handles mountpoint related actions. [state](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-state) has the same options as the [ansible.posix.mount](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html) module provides like `mounted`, `absent`, `present`, `unmounted` and `remounted` (also see [state](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-state)). `state` and `path.name` are required. `path.name` specifies the name of the directory the mountpoint should be mounted. If the directory doesn't exists already it will be created. For further options see next example.

As previous example but the filesystem now gets an label `mnt1`. Also the filesystem gets mounted at `/mnt1` with the `noatime` filesystem option. Also the filesystem gets mounted by using the `mnt1` label as mentioned (`src: LABEL=mnt1`). The `owner` and `group` of the `/mnt1` directory will be `vagrant` and the directory permissions will be `0750`:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
    lvm_lvs:
      - lvname: test-lv-01
        size: 10%VG
        state: present
        fs:
          type: ext4
          opts: -L mnt1
          state: present
          mountpoint:
            opts: noatime
            state: mounted
            src: LABEL=mnt1
            path:
              name: /mnt1
              mode: '0750'
              owner: vagrant
              group: vagrant
```

By default `mountpoint.src` is `/dev/ + vgname + / + lvname`. In the example above `LABEL=mnt1` was used. In `opts: -L mnt` the option was provided to create a filesystem with that label. So that one can be used later for `mountpoint.src`.

`mountpoint` supports the following additional parameters:

- [opts](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-opts)
- [backup](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-backup)
- [dump](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-dump)
- [passno](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-passno)
- [fstab](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-fstab)

Create two volume groups (VG) called `test-vg-01` and `test-vg-02` which uses device `/dev/vdb` and `/dev/vdc` respectively as physical volumes:

```yaml
lvm_vgs:
  - vgname: test-vg-01
    pvs: /dev/vdb
    state: present
  - vgname: test-vg-02
    pvs: /dev/vdc
    state: present
```

Here is an example to unmount a mountpoint and delete its entry from `/etc/fstab`:

```yaml
mountpoint:
  state: absent
  path:
    name: /vg01lv01
```

This of course only works if the mountpoint isn't used by a programm or service.

As deleting filesystems, logical volumes (LV) and volume groups (VG) potentially can destroy your data extra caution should be taken! In this case specifing `state: absent` to delete such a resource isn't enough. `force: true` is also needed. Of course deleting a volume group that still contains a logical volume will fail. Also deleting a logical volume with a filesystem with `state: present` will fail.

The following example will unmount the specified mountpoint (`/vg02lv01`) and delete its entry from `/etc/fstab`, will wipe the `ext4`  filesystem, deletes the logical volume (`vg02lv01`) and finally deletes the volume group (`vg02`) (in that order):

```yaml
lvm_vgs:
  - vgname: vg02
    pvs: /dev/vdc
    state: absent
    force: true
    lvm_lvs:
      - lvname: vg02lv01
        size: 10%VG
        state: absent
        force: true
        fs:
          type: ext4
          state: absent
          force: true
          mountpoint:
            state: absent
            path:
              name: /vg02lv01
```

To be able to create filesystems additional packages are needed. For `ext(2|3|4)` you need `e2fsprogs` package installed e.g. So depending which filesystem you want to setup you may need to install additional package(s) or remove package(s) not needed from the list. Of course you can add whatever packages you want to install to the list

By default only tools needed for `ext(2|3|4)` or `xfs` are installed. For other filesystems you may need `btrfsprogs/btrfs-progs`, `dosfstools`, `f2fs-tools` or `reiserfsprogs` e.g.:

```yaml
# Additional packages for SuSE compatible OSes
additional_packages_suse:
  - e2fsprogs
  - xfsprogs

# Additional packages for Debian compatible OSes
additional_packages_debian:
  - e2fsprogs
  - xfsprogs

# Additional packages for Redhat compatible OSes
additional_packages_redhat:
  - e2fsprogs
  - xfsprogs

# Additional packages for Archlinux compatible OSes
additional_packages_arch:
  - e2fsprogs
  - xfsprogs
```

In general most options of the following modules are supported:

[community.general.lvg](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html)  
[community.general.lvol](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html)  
[community.general.filesystem](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html)  
[ansible.posix.mount](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html)  

Dependencies
------------

This role depends on a few Ansible modules:

[community.general.lvg](https://docs.ansible.com/ansible/latest/collections/community/general/lvg_module.html)  
[community.general.lvol](https://docs.ansible.com/ansible/latest/collections/community/general/lvol_module.html)  
[community.general.filesystem](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html)  
[ansible.posix.mount](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html)  

TODO
----

Currently the following features are not implemented:

[] Volume Group resize  
[] Logical Volume resize/shrink  
[] Logical Volume snapshot  
[] Logical Volume/Filesystem resize  

Example Playbook
----------------

Example 1 (without role tag):

```yaml
- hosts: your-host
  roles:
    - githubixx.lvm
```

Example 2 (assign tag to role):

```yaml
-
  hosts: your-host
  roles:
    -
      role: githubixx.lvm
      tags: role-lvm
```

More examples
-------------

There are a few more examples used for testing this role. See [molecule](https://github.com/githubixx/ansible-role-lvm/tree/master/molecule) directories.

Testing
-------

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-lvm/tree/master/molecule/kvm).

Afterwards molecule can be executed:

```bash
molecule converge -s kvm
```

This will setup quite a few virtual machines (VM) with different supported Linux operating systems and creates various LVM resources.

To clean up run

```bash
molecule destroy -s kvm
```

License
-------

[GNU General Public License v3.0 or later](https://spdx.org/licenses/GPL-3.0-or-later.html)

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
