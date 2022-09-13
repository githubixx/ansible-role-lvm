Changelog
---------

**0.3.0**

- fix various `ansible-lint` issues
- add Github release action to push new release to Ansible Galaxy
- `min_ansible_version` variable value should be string / versions should be string in `meta/main.yml`

**0.2.0**

- fix error when creating only a volume group without a logical volume
- add Molecule test for VG, VG+LV, VG+LV+FS, VG+LV+FS+MOUNT
- add Molecule LVM mirror example/test

**0.1.1**

- initial commit
