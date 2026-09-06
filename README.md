# centos7-lvm-storage-management
Hands-on Linux LVM project on CentOS 7 covering disk partitioning, Physical Volumes (PV), Volume Groups (VG), Logical Volumes (LV), EXT4 filesystem, persistent mounting, LV extension, and LVM snapshots.
# CentOS 7 LVM Storage Management

This project demonstrates how to configure and manage Linux Logical Volume Manager (LVM) storage on a CentOS 7 virtual machine.

## Step 1: Adding a New Hard Disk to the VM

A new virtual hard disk was added to the CentOS 7 virtual machine.

* **Disk:** `/dev/sdb`
* **Capacity:** 2 GiB
* **Disk Type:** VMware Virtual Disk / SCSI block device

The existing `/dev/sda` disk was kept unchanged because it contains the operating system.

---

## Step 2: Inspecting System Block Devices

The `lsblk` command was used to identify the newly added disk and inspect the existing storage layout.

```bash
lsblk
```

The system contained:

* `/dev/sda` — Primary OS disk (20 GiB)
* `/dev/sda1` — `/boot` partition
* `/dev/sda2` — Existing LVM partition
* `/dev/sdb` — New 2 GiB disk with no partitions

Example:

```text
NAME        SIZE TYPE MOUNTPOINT
sda          20G disk
├─sda1        1G part /boot
└─sda2       19G part
  ├─root     ...
  └─swap     ...
sdb          2G disk
```

---

## Step 3: Partitioning the New Disk

The new disk was partitioned using `fdisk`.

```bash
fdisk /dev/sdb
```

Inside `fdisk`:

```text
n
p
1
Enter
Enter
t
8e
w
```

The following operations were performed:

1. Created a new primary partition `/dev/sdb1`.
2. Changed the partition type using `t`.
3. Used partition type code `8e` for Linux LVM.
4. Used `w` to save the partition table.

The partition was verified using:

```bash
fdisk -l /dev/sdb
```

---

## Step 4: Creating the Physical Volume

The new partition was initialized as an LVM Physical Volume (PV).

```bash
pvcreate /dev/sdb1
```

The PV was verified using:

```bash
pvs
```

Detailed information can also be viewed with:

```bash
pvdisplay
```

### LVM Structure

```text
/dev/sdb
    │
    └── /dev/sdb1
          │
          └── Physical Volume (PV)
```

---

## Step 5: Creating the Volume Group

A Volume Group named `vg_data` was created using the Physical Volume.

```bash
vgcreate vg_data /dev/sdb1
```

Verify the Volume Group:

```bash
vgs
```

or:

```bash
vgdisplay
```

The structure is now:

```text
/dev/sdb1
    │
    └── PV
         │
         └── vg_data
              └── Volume Group
```

---

## Step 6: Creating the Logical Volume

A 5 GiB Logical Volume named `lv_data` was created inside `vg_data`.

```bash
lvcreate -L +400M  -n lv_data vg_data
```

Verify the Logical Volume:

```bash
lvs
```

or:

```bash
lvdisplay
```

The complete LVM structure is now:

```text
/dev/sdb
    │
    └── /dev/sdb1
          │
          └── PV
               │
               └── vg_data
                    │
                    └── lv_data
```

---

## Step 7: Creating the XFS Filesystem

An  ext4  filesystem was created on the Logical Volume.

```bash
mkfs.xfs /dev/vg_data/lv_data
```

The filesystem was verified using:

```bash
blkid /dev/vg_data/lv_data
```

Expected filesystem type:

```text
TYPE="ext4"
```

---

## Step 8: Creating the Mount Point

A directory was created to use as the mount point.

```bash
mkdir /root/Projects
```

The Logical Volume was mounted:

```bash
mount    /dev/vg_data/lv_project    /root/Projects```

Verify the mount:

```bash
df -h
```

or:

```bash
lsblk
```

---

## Step 9: Configuring Persistent Mounting

To make the filesystem mount automatically after a reboot, an entry was added to `/etc/fstab`.

First, the UUID was obtained:

```bash
blkid /dev/vg_data/lv_project
```

Then `/etc/fstab` was edited:

```bash
vi /etc/fstab
```

Example entry:

```text
/dev/sdb1    /root/Projects    ext4    defaults    0 0
```

Using the UUID is recommended because it uniquely identifies the filesystem.

---

## Step 10: Testing the `/etc/fstab` Configuration

Before rebooting, the `/etc/fstab` configuration was tested using:

```bash
umount /root/Projects
```

Then:

```bash
mount -a
```

Verify:

```bash
df -h /root/Projects
```

If `/root/Projects` is mounted successfully, the `/etc/fstab` configuration is working correctly.

---

## Step 11: Testing Data Storage

A test directory and file were created:

```bash
mkdir /root/Projects/project
```

```bash
echo "LVM Storage Management Project" > /mnt/data/project/test.txt
```

Verify the file:

```bash
cat /mnt/data/project/test.txt
```

This confirms that the Logical Volume is available for normal file operations.

---

## Final LVM Architecture

```text
                    CentOS 7
                       │
                  /dev/sdb
                  2 GiB Disk
                       │
                       ▼
                  /dev/sdb1
                       │
                       ▼
              Physical Volume
                       │
                       ▼
                   vg_data
                Volume Group
                       │
                       ▼
                   lv_data
              Logical Volume
                   400 MiB
                       │
                       ▼
                     EXT4
                       │
                       ▼
                  /root/Projects

```

## Verification Commands

The following commands can be used to verify the complete LVM configuration:

```bash
lsblk
pvs
vgs
lvs
blkid
df -h
mount | grep /root/Projects
```

## Skills Demonstrated

* Linux disk management
* `fdisk` partitioning
* LVM Physical Volume management
* LVM Volume Group management
* Logical Volume creation
* XFS filesystem management
* Filesystem mounting
* `/etc/fstab` configuration
* Storage verification
* Basic Linux storage troubleshooting
