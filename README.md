Ansible Role: LUKS Disks
=========

An Ansible role that configures disks with LUKS containers and filesystems, primarily for use in a NAS or homelab.

Requirements
------------

None

Role Variables
--------------

See `defaults/main.yml`.

Dependencies
------------

Relies on Ansible community collections:
* community.crypto.luks_device
* community.general.filesystem
* community.general.crypttab

Example Playbook
----------------

### provision-disks-luks.yml

This playbook creates a keyfile on the remote machine, copies it to the source machine as a backup, matches the expected serial numbers for each disk, creates and configures LUKS containers, creates and configures ext4 filesystems on each disk, and configures automatic decryption with crypttab, and automatic filesystem mounting with fstab.

```
---
- name: Configure LUKS disks
  hosts: all
  become: yes

  roles:
    - role: jhaycr.luks_disks
  
  vars:
    luks_keyfile_path: /etc/luks.key
    luks_keyfile_local_backup_path: "~/.luks.key"

    luks_disks_validate_serial: true
    luks_disks:
      - name: "parity1"
        device: "/dev/disk/by-id/ata-MODEL_ABC123"
        serial: "ABC123"
        fstype: ext4
      - name: "data1"
        device: "/dev/disk/by-id/ata-MODEL_DEF456"
        serial: "DEF456"
        fstype: ext4
      - name: "data2"
        device: "/dev/disk/by-id/ata-MODEL_GHI789"
        serial: "GHI789"
        fstype: ext4
```

### provision-disks.yml

This playbook extends the above as part of a larger disk provisioning strategy. I like to use [snapRAID](https://github.com/amadvance/snapraid) and [mergerfs](https://github.com/trapexit/mergerfs).

```
---
- name: Configure LUKS disks, Snapraid, mergerfs
  hosts: all
  become: yes

  roles:
    - role: jhaycr.luks_disks
    - role: geerlingguy.docker
    - role: IronicBadger.snapraid
    - role: tigattack.mergerfs
  
  vars:
    luks_keyfile_path: /etc/luks.key
    luks_keyfile_local_backup_path: "~/.luks.key"

    luks_disks_validate_serial: true
    luks_disks:
      - name: "parity1"
        device: "/dev/sdc"
        serial: "ABC123"
        fstype: ext4
      - name: "data1"
        device: "/dev/sdd"
        serial: "DEF456"
        fstype: ext4
      - name: "data2"
        device: "/dev/sde"
        serial: "GHI789"
        fstype: ext4

    snapraid_install: true
    snapraid_runner: true

    snapraid_data_disks:
      - path: /mnt/data1
        content: true
      - path: /mnt/data2
        content: true

    snapraid_parity_disks:
      - path: /mnt/parity1
        content: false

    mergerfs_mounts:
      - path: /mnt/storage
        branches:
          - /mnt/data*
        options: defaults,nonempty,allow_other,use_ino,cache.files=off,moveonenospc=true,category.create=mfs,dropcacheonclose=true,minfreespace=250G,fsname=mergerfs
```

License
-------

MIT

Author Information
------------------

Josh Haycraft