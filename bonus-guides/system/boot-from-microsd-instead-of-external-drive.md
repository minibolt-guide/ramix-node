---
layout:
  width: wide
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
  metadata:
    visible: true
---

# Boot from microSD instead of an external drive

If the Raspberry Pi cannot boot from your external drive, you can boot from a microSD card and store all the application data on the external drive.

{% hint style="warning" %}
Difficulty: Medium
{% endhint %}

There are a few additional steps compared to the default RaMiX guide to boot from a microSD card and store the data on an external drive.

When writing Raspberry Pi OS to the boot medium, use a **high endurance microSD card of 16+ GB** instead of the external drive.

## System configuration <a href="#system-configuration" id="system-configuration"></a>

1. Connect your external drive to the Raspberry Pi using one of the <mark style="color:blue;">**blue**</mark>**&#x20;USB 3.0 ports.**
2. Follow the [Configuration](../../index-1/configuration.md) section until you reach the [Data directory](../../index-1/configuration.md#data-directory) section, which we will skip and continue with the steps below.

{% hint style="info" %}
**Remember:** if your external drive performs poorly during the [Check Drive Performance](../../index-1/configuration.md#check-drive-performance) step, refer to the [Fix Bad USB3 performance](../troubleshooting.md#fix-bad-usb3-performance) instructions in the guide
{% endhint %}

## **Format external drive**

Now, we will condition the external drive to create partitions. As a server installation, the Linux native file system Ext4 is the best choice for the external hard disk.

* List all block devices with additional information. The list shows the devices (e.g. `sda`) and the partitions they contain (e.g. `sda1`)

```bash
lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE,SIZE,LABEL,MODEL
```

**Example** of expected output without existing partitions:

```
NAME        MOUNTPOINT      UUID                                 FSTYPE     SIZE LABEL  MODEL
sda                                                                       931.5G        Ext_SSD
mmcblk0                                                                      58G
├─mmcblk0p1 /boot/firmware  4EF5-6F55                            vfat       512M bootfs
└─mmcblk0p2 /               ce208fd3-38a8-424a-87a2-cd44114eb820 ext4      57.5G rootfs
```

**Example** of expected output with existing partitions:

<pre><code>NAME        MOUNTPOINT      UUID                                 FSTYPE     SIZE LABEL  MODEL
sda                                                                       931.5G        Ext_SSD
└─<a data-footnote-ref href="#user-content-fn-1">sda1</a>                      3aab0952-3ed4-4652-b203-d994c4fdff20 ext4     931.5G
mmcblk0                                                                      58G
├─mmcblk0p1 /boot/firmware  4EF5-6F55                            vfat       512M bootfs
└─mmcblk0p2 /               ce208fd3-38a8-424a-87a2-cd44114eb820 ext4      57.5G rootfs
</code></pre>

* Type this command to use the `"fdisk"` utility and manage the disk

```bash
sudo fdisk /dev/sda
```

-> **2 cases**, depending on whether your drive contains partitions or not:

{% tabs %}
{% tab title="Case 1: doesn't contain existing partitions" %}
If you didn't see any "sda**X**" partition in the previous step, i.e `sda1`:

* Press **`"n"`** to create a new partition and then press ENTER until the prompt shows you:&#x20;

```
Created a new partition X of type 'Linux filesystem'
(Command (m for help)) again
```
{% endtab %}

{% tab title="Case 2: contain existing partitions" %}
If you did see an existing partition "sda**X**" in the previous step, i.e `sda1`:

* Press **`"d"`** to delete the existing partitions and then press ENTER until the prompt shows you:

```
Partition X has been deleted
(Command (m for help)) again
```

{% hint style="info" %}
If you have more than one partition, repeat the before step until there are none left
{% endhint %}

* Press **`"n"`** to create a new partition and then press ENTER until the prompt shows:

```
Created a new partition X of type 'Linux filesystem'
(Command (m for help)) again
```
{% endtab %}
{% endtabs %}

-> Finally, don't forget, to type **`w`**  and **ENTER** to write table to disk and exit

{% hint style="info" %}
This will create a new partition called probably **`"sda1"` -> Take note of this reference for the next step**
{% endhint %}

* Format the partition on the external drive with the Ext4 system file (replace`[NAME]` to your partition name, e.g. `sda1`)

```bash
sudo mkfs.ext4 /dev/[NAME]
```

{% hint style="danger" %}
**Attention: this will delete all existing data on the external drive!**
{% endhint %}

**Example** of command:

```bash
sudo mkfs.ext4 /dev/sda1
```

**Example** of expected output:

<pre><code>mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1572608 4k blocks and 393216 inodes
Filesystem UUID: <a data-footnote-ref href="#user-content-fn-2">dafc3c67-c6e5-4eaa-8840-adaf604c85db</a>
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736
</code></pre>

{% hint style="info" %}
Take note of the `Filesystem UUID` -> i.e: _dafc3c67-c6e5-4eaa-8840-adaf604c85db_, you will need this more later
{% endhint %}

### **Mount external drive**

The external drive is then attached to the file system and becomes available as a regular folder (this is called “mounting”).

* List the block devices one more time to ensure that UUID has been assigned

```bash
lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE,SIZE,LABEL,MODEL
```

Example of expected output:

<pre><code>NAME        MOUNTPOINT      UUID                                         FSTYPE       SIZE LABEL  MODEL
sda                                                                                 931.5G        Ext_SSD
└─sda1                      <a data-footnote-ref href="#user-content-fn-3">3aab0952-3ed4-4652-b203-d994c4fdff20</a>      ext4     931.5G
mmcblk0                                                                                58G
├─mmcblk0p1 /boot/firmware  4EF5-6F55                                       vfat      512M  bootfs
└─mmcblk0p2 /               ce208fd3-38a8-424a-87a2-cd44114eb820            ext4      57.5G rootfs
</code></pre>

{% hint style="info" %}
Copy the new partition `UUID` into a text editor on your regular machine
{% endhint %}

* Edit the `fstab` file

```bash
sudo nano /etc/fstab
```

* Add the following as a new line **at the end of the file**

<pre><code>UUID=<a data-footnote-ref href="#user-content-fn-4">&#x3C;yourUUID></a> /data ext4 rw,nosuid,dev,noexec,noatime,nodiratime,auto,nouser,async,nofail 0 2
</code></pre>

{% hint style="info" %}
Replace `<yourUUID>` with your `UUID` obtained before
{% endhint %}

* Create the data directory as a mount point

```bash
sudo mkdir /data
```

* Assing to the `admin` user as the owner of the **`/data`** folder

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

* Check the file system. Is `/data` listed?

```bash
df -h /data
```

**Example** of expected output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       938G   77M  891G   1% /data
```

Or check the mount point using `lsblk`

```bash
lsblk -o NAME,MOUNTPOINT,UUID,FSTYPE,SIZE,LABEL,MODEL
```

Example of expected output:

<pre><code>NAME        MOUNTPOINT     UUID                                 FSTYPE   SIZE LABEL  MODEL
sda                                                                    931.5G        Ext_SSD
└─sda1      <a data-footnote-ref href="#user-content-fn-5">/data</a>          15af9b1d-ca7c-441f-b101-c1a0cf76a062 ext4   931.5G
mmcblk0                                                                   58G
├─mmcblk0p1 /boot/firmware 4EF5-6F55                            vfat     512M bootfs
└─mmcblk0p2 /              ce208fd3-38a8-424a-87a2-cd44114eb820 ext4    57.5G rootfs
</code></pre>

### **Resize & move the swap file to the new drive**

The swap file acts as slower memory and is essential for system stability. microSD cards are not performant and degrade over time under constant read/write activity. Therefore, we move the swap file to the external drive and increase its size.

* Edit the `dphys-swapfile` configuration file

```bash
sudo nano -l +12 /etc/dphys-swapfile
```

* Uncomment the entry `CONF_SWAPSIZE` by deleting the `#` in front of it, and editing to match this

```
CONF_SWAPFILE=/data/swapfile
```

* Comment on the entry `CONF_SWAPSIZE` by placing a "`#"` symbol in front of it. Save and exit

```
#CONF_SWAPSIZE=512
```

{% hint style="info" %}
It will then be created dynamically to use ½ of the total RAM size installed
{% endhint %}

* Recreate and activate new swapfile

```bash
sudo systemctl restart dphys-swapfile
```

* Check the new swap size

```bash
swapon --show
```

Example of expected output:

```
NAME           TYPE SIZE USED PRIO
/data/swapfile file   2G   0B   -2
```

## Continue with the guide <a href="#continue-with-the-guide" id="continue-with-the-guide"></a>

{% hint style="success" %}
That’s it: your Raspberry Pi now boots from the microSD card while the data directory `/data` is located on the external drive.

-> You can now continue with the [Security](../../index-1/security.md) section <-
{% endhint %}

[^1]: Existing partition

[^2]: Take note

[^3]: Take note of this

[^4]: Replace this

[^5]: Check this
