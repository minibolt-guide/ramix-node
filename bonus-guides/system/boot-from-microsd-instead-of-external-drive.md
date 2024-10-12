---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Boot from microSD instead of external drive

If the Raspberry Pi is not able to boot from your external drive, you can boot from a microSD card and use the external drive to store all the application data.

{% hint style="danger" %}
Status: Not tested on RaMiX
{% endhint %}

{% hint style="success" %}
Difficulty: Easy
{% endhint %}

#### Steps required <a href="#steps-required" id="steps-required"></a>

To boot from a microSD card and store the data on an external drive, there are a few additional steps compared to the default RaMiX guide. Below is a summary of the main differences, with detailed guidance in the following sections.

1. [Operating system](../../index-1/operating-system.md):
   * write the operating system to the microSD card instead of the external drive
2. [Configuration](../../index-1/configuration.md):
   * attach the external drive
   * test the USB3 performance
   * format the drive
   * mount the drive to `/data`

***

#### Operating system <a href="#operating-system" id="operating-system"></a>

When writing Raspberry Pi OS to the boot medium, use a high-quality microSD card of 8+ GB instead of the external drive.

***

## System configuration <a href="#system-configuration" id="system-configuration"></a>

Connect your external drive to the Raspberry Pi using one of the blue USB3 ports.

Follow the [Configuration](../../index-1/configuration.md) section until you reach the [Data directory](../../index-1/configuration.md#data-directory) section, continuing with the instructions below.

In case your external drive shows poor performance, follow the [Fix bad USB3 performance](../../troubleshooting.md#fix-bad-usb3-performance) instructions, as mentioned in the guide.

### **Format external drive**

We will now format the external drive. As a server installation, the Linux native file system Ext4 is the best choice for the external hard disk.

* List all block devices with additional information. The list shows the devices (e.g. `sda`) and the partitions they contain (e.g. `sda1`)

```bash
lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE,SIZE,LABEL,MODEL
```

Example of expected output:

```
NAME        MOUNTPOINT UUID                                 FSTYPE   SIZE LABEL  MODEL
sda                                                                931.5G        Ext_SSD
└─sda1                 3aab0952-3ed4-4652-b203-d994c4fdff20 ext4   931.5G
mmcblk0                                                             14.8G
|-mmcblk0p1 /boot      DBF3-0E3A                            vfat     256M boot
`-mmcblk0p2 /          b73b1dc9-6e12-4e68-9d06-1a1892663226 ext4    14.6G rootfs
```

* If your drive does not contain any partitions, follow this [How to Create a Disk Partitions in Linux](https://www.tecmint.com/create-disk-partitions-in-linux/) guide first.
* Make a note of the partition name of your external drive (in this case “sda1”).
* Format the partition on the external drive with Ext4 (use `[NAME]` from above, e.g. `sda1`)

{% hint style="danger" %}
**Attention: this will delete all existing data on the external drive!**
{% endhint %}

```bash
sudo mkfs.ext4 /dev/[NAME]
```

### &#x20;**Mount external drive**

The external drive is then attached to the file system and becomes available as a regular folder (this is called “mounting”).

* List the block devices once more and copy the new partition’s `UUID` into a text editor on your main machine

```bash
lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE,SIZE,LABEL,MODEL
```

Example of expected output:

```
NAME        MOUNTPOINT UUID                                 FSTYPE   SIZE LABEL  MODEL
sda                                                                931.5G        Ext_SSD
└─sda1                 3aab0952-3ed4-4652-b203-d994c4fdff20 ext4   931.5G
mmcblk0                                                             14.8G
|-mmcblk0p1 /boot      DBF3-0E3A                            vfat     256M boot
`-mmcblk0p2 /          b73b1dc9-6e12-4e68-9d06-1a1892663226 ext4    14.6G rootfs
```

* Edit the `fstab` file

```bash
sudo nano /etc/fstab
```

* Add the following as a new line at the end, replacing `123456` with your own `UUID`

```
UUID=123456 /data ext4 rw,nosuid,dev,noexec,noatime,nodiratime,auto,nouser,async,nofail 0 2
```

{% hint style="info" %}
_more:_ [_complete fstab guide_](https://linuxconfig.org/how-fstab-works-introduction-to-the-etc-fstab-file-on-linux)
{% endhint %}

* Create the data directory as a mount point

```bash
sudo mkdir /data
```

* Assing to the `admin` user as the owner of the **`(/data)`** folder

```bash
sudo chown admin:admin /data
```

* Make the directory immutable to prevent data from being written on the microSD card if the external drive is not mounted

```bash
sudo chattr +i /data
```

* Mount all drives

```bash
sudo mount -a
```

* Check the file system. Is “/data” listed?

```bash
df -h /data
```

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       938G   77M  891G   1% /data
```

### **Move swap file to New Drive**

The swap file acts as slower memory and is essential for system stability. MicroSD cards are not very performant and degrade over time under constant read/write activity. Therefore, we move the swap file to the external drive and increase its size as well.

* Edit the `dphys-swapfile` configuration file

```bash
sudo nano -l +12 /etc/dphys-swapfile
```

* Uncomment the entry `CONF_SWAPSIZE` by deleting the `#` in front of it, and editing to match this

```
CONF_SWAPFILE=/data/swapfile
```

* Comment the  `CONF_SWAPSIZE` by placing a `#` in front of it

```
#CONF_SWAPSIZE=200
```

* Recreate and activate new swapfile

```bash
sudo systemctl restart dphys-swapfile
```

***

## &#x20;Continue with the guide <a href="#continue-with-the-guide" id="continue-with-the-guide"></a>

That’s it: your Raspberry Pi now boots from the microSD card while the data directory `/data` is located on the external drive.

You can now continue with the [Remote access](../../index-1/remote-access.md) section
