# Linux LVM Storage Management – Commands

This document contains the commands used to configure and verify LVM storage.

---

## 1. Check Available Disks

Use `lsblk` to display block devices.

```bash
lsblk
```

Example:

```text
NAME        SIZE TYPE MOUNTPOINT
sda          20G disk
├─sda1        1G part /boot
└─sda2       19G part
sdb          10G disk
```

Here, `/dev/sdb` is the newly added disk.

---

## 2. Partition the New Disk

Start `fdisk`:

```bash
fdisk /dev/sdb
```

Inside `fdisk`:

```text
n
```

Create a new partition.

Then change the partition type:

```text
t
```

For older MBR partition tables, use:

```text
8e
```

This represents Linux LVM.

Finally, save the changes:

```text
w
```

The resulting partition is:

```text
/dev/sdb1
```

> Note: On modern GPT partition tables, the exact partition-type code can differ. The important point is to select the Linux LVM partition type offered by your `fdisk` version.

---

## 3. Verify the Partition

Run:

```bash
lsblk
```

You should see:

```text
sdb
└─sdb1
```

You can also check the partition table with:

```bash
fdisk -l /dev/sdb
```

---

## 4. Create Physical Volume

Initialize the partition for LVM:

```bash
pvcreate /dev/sdb1
```

Verify:

```bash
pvs
```

Detailed information:

```bash
pvdisplay
```

---

## 5. Create Volume Group

Create a Volume Group named `vg_data`:

```bash
vgcreate vg_data /dev/sdb1
```

Verify:

```bash
vgs
```

Detailed information:

```bash
vgdisplay vg_data
```

---

## 6. Create Logical Volume

Create a Logical Volume using all available space:

```bash
lvcreate -l 100%FREE -n lv_data vg_data
```

Verify:

```bash
lvs
```

Detailed information:

```bash
lvdisplay /dev/vg_data/lv_data
```

---

## 7. Format the Logical Volume

Create an XFS filesystem:

```bash
mkfs.xfs /dev/vg_data/lv_data
```

The device can also be referenced as:

```bash
/dev/mapper/vg_data-lv_data
```

---

## 8. Create Mount Point

Create the directory:

```bash
mkdir -p /mnt/data
```

---

## 9. Mount the Logical Volume

Mount the filesystem:

```bash
mount /dev/vg_data/lv_data /mnt/data
```

Check the mount:

```bash
df -h
```

Or:

```bash
lsblk
```

---

## 10. Configure /etc/fstab

Open the file:

```bash
vi /etc/fstab
```

Add:

```text
/dev/mapper/vg_data-lv_data  /mnt/data  xfs  defaults  0 0
```

Save the file.

---

## 11. Test /etc/fstab

Run:

```bash
mount -a
```

If there is no error, the `/etc/fstab` configuration is valid enough for the mount operation.

Verify:

```bash
df -h /mnt/data
```

---

## 12. Check Filesystem

Use:

```bash
df -Th
```

This displays the filesystem type.

Example:

```text
Filesystem                     Type  Size  Used Avail Use% Mounted on
/dev/mapper/vg_data-lv_data    xfs    10G   ...   ...   ... /mnt/data
```

---

## 13. Check LVM Information

### Physical Volumes

```bash
pvs
pvdisplay
```

### Volume Groups

```bash
vgs
vgdisplay
```

### Logical Volumes

```bash
lvs
lvdisplay
```

---

## 14. Complete Verification

Run:

```bash
lsblk
pvs
vgs
lvs
df -h
```

These commands verify the complete LVM configuration.

---

## 15. Useful LVM Management Commands

### Extend a Logical Volume

Example:

```bash
lvextend -L +2G /dev/vg_data/lv_data
```

For XFS, grow the filesystem after extending the LV:

```bash
xfs_growfs /mnt/data
```

Verify:

```bash
df -h /mnt/data
```

### Extend a Volume Group

If another disk/partition is available:

```bash
pvcreate /dev/sdc1
vgextend vg_data /dev/sdc1
```

Check:

```bash
vgs
```

---

## 16. Remove LVM Configuration

**Use these commands only when you intentionally want to delete the storage configuration.**

Unmount:

```bash
umount /mnt/data
```

Remove Logical Volume:

```bash
lvremove /dev/vg_data/lv_data
```

Remove Volume Group:

```bash
vgremove vg_data
```

Remove Physical Volume:

```bash
pvremove /dev/sdb1
```

The partition can then be removed using:

```bash
fdisk /dev/sdb
```

> Warning: Removing an LV, VG, or PV can permanently destroy data. Always verify the device name before running destructive commands.
