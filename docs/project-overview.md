# Linux LVM Storage Management – Project Overview

## 1. Project Description

This project demonstrates how to add a new virtual hard disk to a Linux virtual machine and configure it using **Logical Volume Manager (LVM)**.

A 10 GiB virtual disk was added to the VMware virtual machine. The disk was partitioned, converted into an LVM Physical Volume, added to a Volume Group, and then used to create a Logical Volume.

The Logical Volume was formatted with the **XFS filesystem** and configured to mount automatically using `/etc/fstab`.

## 2. Objective

The main objectives of this project are:

* Add a new virtual disk to a Linux system.
* Identify the newly added disk.
* Create a partition using `fdisk`.
* Configure the partition for LVM.
* Create a Physical Volume (PV).
* Create a Volume Group (VG).
* Create a Logical Volume (LV).
* Format the Logical Volume with XFS.
* Create a mount point.
* Configure persistent mounting using `/etc/fstab`.
* Verify the complete storage configuration.

## 3. Environment

| Component        | Configuration |
| ---------------- | ------------- |
| Virtualization   | VMware        |
| Operating System | Linux         |
| New Disk         | `/dev/sdb`    |
| Disk Size        | 10 GiB        |
| Partition        | `/dev/sdb1`   |
| Volume Group     | `vg_data`     |
| Logical Volume   | `lv_data`     |
| Filesystem       | XFS           |
| Mount Point      | `/mnt/data`   |

## 4. Storage Architecture

The storage configuration follows this structure:

```text
10 GiB Virtual Disk
       │
       ▼
    /dev/sdb
       │
       ▼
    /dev/sdb1
       │
       ▼
Physical Volume (PV)
       │
       ▼
Volume Group (vg_data)
       │
       ▼
Logical Volume (lv_data)
       │
       ▼
   XFS Filesystem
       │
       ▼
    /mnt/data
```

## 5. Implementation Steps

### Step 1 – Add New Disk

A new 10 GiB virtual disk was attached to the Linux virtual machine.

Disk:

```text
/dev/sdb
```

### Step 2 – Verify the Disk

The `lsblk` command was used to identify the new disk.

```bash
lsblk
```

The new disk appeared as `/dev/sdb`.

### Step 3 – Create Partition

The `fdisk` utility was used to create a partition.

```bash
fdisk /dev/sdb
```

A new partition was created:

```text
/dev/sdb1
```

The partition type was changed to Linux LVM.

### Step 4 – Create Physical Volume

The partition was initialized as an LVM Physical Volume.

```bash
pvcreate /dev/sdb1
```

### Step 5 – Create Volume Group

A Volume Group named `vg_data` was created.

```bash
vgcreate vg_data /dev/sdb1
```

### Step 6 – Create Logical Volume

A Logical Volume named `lv_data` was created.

```bash
lvcreate -l 100%FREE -n lv_data vg_data
```

### Step 7 – Create Filesystem

The Logical Volume was formatted with XFS.

```bash
mkfs.xfs /dev/vg_data/lv_data
```

### Step 8 – Create Mount Point

A directory was created for mounting the filesystem.

```bash
mkdir -p /mnt/data
```

### Step 9 – Configure Persistent Mount

The Logical Volume was added to `/etc/fstab`.

Example:

```text
/dev/mapper/vg_data-lv_data  /mnt/data  xfs  defaults  0 0
```

### Step 10 – Mount and Verify

The filesystem was mounted using:

```bash
mount -a
```

The configuration was verified using:

```bash
lsblk
df -h
```

## 6. Final Result

A 10 GiB virtual disk was successfully configured using LVM.

The final storage hierarchy was:

```text
/dev/sdb
   └── /dev/sdb1
          └── PV
                └── vg_data
                      └── lv_data
                            └── XFS
                                  └── /mnt/data
```

The storage is configured to mount automatically after system reboot using `/etc/fstab`.

## 7. Skills Demonstrated

* Linux Storage Management
* LVM Administration
* Disk Partitioning
* Filesystem Management
* XFS
* `/etc/fstab` configuration
* Mounting and unmounting filesystems
* VMware Virtual Machine Management
* Linux troubleshooting and verification
