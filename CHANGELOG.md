<!--
Copyright (C) 2021-2024 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

# Changelog

## 0.4.0

- add resize filesystem `resizefs` item
- remove support for Ubuntu 18.04 (reached EOL)
- remove support for Debian 10 (reached EOL)
- remove support for Fedora 35/36 (reached EOL)
- remove support for openSUSE Leap 15.3 (reached EOL)
- add support for Fedora 39
- add support for Debian 12
- add support for openSUSE Leap 15.4
- add support for openSUSE Leap 15.5
- add support for Rocky Linux 8/9
- add support for Almalinux 8/9
- update Github workflow
- Molecule: change IP addresses
- Molecule: fix ansible-lint issues

## 0.3.0

- fix various `ansible-lint` issues
- update Molecule scenario `kvm-ubuntu-20-only`
- update Molecule scenario `kvm-lvm-mirror`
- update Molecule scenario `kvm`
- `tasks/main.yml`: use fully qualified Ansible module names
- remove support for Fedora 33 + 34
- add support for Fedora 35 + 36
- remove support for opensuse 15.2
- add support for opensuse 15.3
- add support for Ubuntu 22.04
- molecule: add `vars/test-lvm-minimal.yml` with some minimal LVM variables enough to test core functionality
- add Github release action to push new release to Ansible Galaxy
- `min_ansible_version` variable value should be string / versions should be string in `meta/main.yml`

## 0.2.0

- fix error when creating only a volume group without a logical volume
- add Molecule test for VG, VG+LV, VG+LV+FS, VG+LV+FS+MOUNT
- add Molecule LVM mirror example/test

## 0.1.1

- initial commit
