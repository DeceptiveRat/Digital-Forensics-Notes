## 1. *Logical Volume Manager(LVM)*
- storage space is managed by combining or pooling the capacity of the available drives
![[traditional_storage.png]]
![[LVM_storage.png]]

## 2. forensic analysis
1. mount LVM on loopback device
``` sh
sudo losetup -fP image.raw
```

2. scan for new physical volumes
``` sh
sudo lvm pvscan
```

3. activate volume group
``` sh
sudo vgchange -ay $VOLUMEGROUP
```

4. analyze data at `/dev/mapper/$VOLUMEGROUP-$VOLUMENAME`

5. clean up
``` sh
sudo vgchange -an $VOLUMEGROUP
sudo losetup -d /dev/$LOOPNUMBER
```

## references
[1] (Damon Garn, 2020/12/07) Logical Volume Manager (LVM) versus standard partitioning in Linux, 2026/05/27, https://www.redhat.com/en/blog/lvm-vs-partitioning
[2] (Forensics Wiki) Linux logical volume manager (lvm), 2026/05/27, https://forensics.wiki/linux_logical_volume_manager_(lvm)/
