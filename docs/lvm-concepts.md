# Linux LVM Concepts

## 1. What is LVM?

**LVM (Logical Volume Manager)** is a Linux storage management technology that provides flexible management of disk space.

Instead of directly creating filesystems on physical disk partitions, LVM introduces an additional storage layer.

This makes it easier to:

* Increase storage capacity
* Decrease storage capacity when supported by the filesystem
* Create logical volumes
* Combine multiple disks
* Manage storage more flexibly

## 2. LVM Architecture

The basic LVM structure is:

```text
Physical Disk
     │
     ▼
Partition
     │
     ▼
Physical Volume (PV)
     │
     ▼
Volume Group (VG)
     │
     ▼
Logical Volume (LV)
     │
     ▼
Filesystem
     │
     ▼
Mount Point
```

In this project:

```text
/dev/sdb
   │
   ▼
/dev/sdb1
   │
   ▼
PV
   │
   ▼
vg_data
   │
   ▼
lv_data
   │
   ▼
XFS
   │
   ▼
/mnt/data
```

## 3. Physical Volume (PV)

A **Physical Volume** is a disk or partition that has been initialized for use by LVM.

In this project:

```text
/dev/sdb1
```

was converted into a Physical Volume using:

```bash
pvcreate /dev/sdb1
```

To view Physical Volumes:

```bash
pvs
```

or:

```bash
pvdisplay
```

## 4. Volume Group (VG)

A **Volume Group** is a storage pool created from one or more Physical Volumes.

In this project, the Volume Group is:

```text
vg_data
```

It was created using:

```bash
vgcreate vg_data /dev/sdb1
```

To view Volume Groups:

```bash
vgs
```

or:

```bash
vgdisplay
```

A Volume Group provides storage space from which Logical Volumes can be created.

## 5. Logical Volume (LV)

A **Logical Volume** is a virtual storage device created inside a Volume Group.

In this project:

```text
lv_data
```

was created inside:

```text
vg_data
```

Command:

```bash
lvcreate -l 100%FREE -n lv_data vg_data
```

The Logical Volume can then be formatted with a filesystem.

To view Logical Volumes:

```bash
lvs
```

or:

```bash
lvdisplay
```

## 6. Filesystem

A filesystem provides a structure that Linux uses to store and organize files.

This project uses:

```text
XFS
```

The filesystem was created using:

```bash
mkfs.xfs /dev/vg_data/lv_data
```

## 7. Mount Point

A mount point is a directory where a filesystem becomes accessible to users and applications.

In this project:

```text
/mnt/data
```

was used as the mount point.

The directory was created using:

```bash
mkdir -p /mnt/data
```

The filesystem can then be mounted using:

```bash
mount /dev/vg_data/lv_data /mnt/data
```

## 8. /etc/fstab

`/etc/fstab` is a configuration file that defines filesystems that should be mounted automatically.

Example:

```text
/dev/mapper/vg_data-lv_data  /mnt/data  xfs  defaults  0 0
```

The fields mean:

| Field                         | Meaning                  |
| ----------------------------- | ------------------------ |
| `/dev/mapper/vg_data-lv_data` | Device                   |
| `/mnt/data`                   | Mount point              |
| `xfs`                         | Filesystem type          |
| `defaults`                    | Default mount options    |
| `0`                           | Dump setting             |
| `0`                           | Filesystem check setting |

After editing `/etc/fstab`, the configuration can be tested with:

```bash
mount -a
```

## 9. Important LVM Commands

### Physical Volume Commands

```bash
pvcreate
pvs
pvdisplay
pvremove
```

### Volume Group Commands

```bash
vgcreate
vgs
vgdisplay
vgextend
vgreduce
vgremove
```

### Logical Volume Commands

```bash
lvcreate
lvs
lvdisplay
lvextend
lvreduce
lvremove
```

## 10. Advantages of LVM

### Flexible Storage Management

Logical Volumes can be created according to storage requirements.

### Easy Expansion

Logical Volumes can be extended when additional space is available.

Example:

```bash
lvextend
```

### Multiple Physical Disks

Multiple Physical Volumes can be combined into one Volume Group.

```text
Disk 1 ──┐
         ├── Volume Group
Disk 2 ──┘
```

### Better Storage Organization

LVM provides a logical layer between physical storage and filesystems.

## 11. LVM vs Traditional Partitioning

### Traditional Partitioning

```text
/dev/sda1 → Filesystem
/dev/sda2 → Filesystem
/dev/sda3 → Filesystem
```

The partition sizes are relatively fixed.

### LVM

```text
Disk
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
Filesystem
```

LVM provides more flexibility for managing storage.

## 12. Verification Commands

Useful commands for checking the complete configuration:

```bash
lsblk
pvs
vgs
lvs
df -h
mount
```

These commands help a Linux administrator understand the relationship between the disk, partition, LVM components, filesystem, and mount point.


