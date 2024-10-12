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

# 1.4 Configuration

{% hint style="danger" %}
Status: Not tested on RaMiX
{% endhint %}

You are now on the command line of your own Bitcoin node. Let's start with the configuration.

<figure><img src="../.gitbook/assets/configuration.jpg" alt="" width="375"><figcaption></figcaption></figure>

## Enable PCIe (only Raspberry Pi 5 + NVMe case)

{% hint style="info" %}
If you connected the SSD directly to the USB (**more common**), and you won't use M.2 NVMe drive, you can skip these steps and go to the [Install general dependency packages](configuration.md#install-general-dependency-packages) section
{% endhint %}

By default, the PCIe connector is not enabled **unless connected to a** [**HAT+ device**](https://www.raspberrypi.com/products/m2-hat-plus/). To enable the connector, follow these steps. If you connected **a** [**HAT+ device**](https://www.raspberrypi.com/products/m2-hat-plus/), skip this step and go directly to the next [PCIe Gen 3.0 section](configuration.md#pcie-gen-3.0).

* With the user `admin`, edit the `config.txt` file

```bash
sudo nano /boot/firmware/config.txt
```

* Add the following line at the end of the file behind `[all]` section. Save and exit if you will not use PCIe Gen 3.0, if yes, continue with the next step without exiting the editor (you must choose via `config.txt` option in the next step)

```
dtparam=pciex1
```

### PCIe Gen 3.0

{% hint style="danger" %}
The Raspberry Pi 5 is not certified for Gen 3.0 speeds. PCIe Gen 3.0 connections may be unstable
{% endhint %}

Raspberry Pi 5 uses Gen 2.0 speeds (5 GT/s) by default. Apply the next config to force Gen 3.0 (8 GT/s) speeds (if the peripheral board and M.2 NVMe drive support (more common)). 2 different options:

{% tabs %}
{% tab title="via config.txt" %}
* Edit the `config.txt` file

```bash
sudo nano /boot/firmware/config.txt
```

* Add the following line at the end of the file behind the `[all]` section. Save and exit if you will not use PCIe Gen 3.0, if yes, continue with the next step without exiting the editor

```
dtparam=pciex1_gen=3
```
{% endtab %}

{% tab title="via raspi-config" %}
* Run the following command to open the Raspberry Pi Configuration CLI

```bash
sudo raspi-config
```

Complete the following steps to enable PCIe Gen 3.0 speeds:

1. Select `Advanced Options`
2. Select `PCIe Speed`
3. Choose `Yes` to enable PCIe Gen 3 mode
4. Select `Finish` to exit
{% endtab %}
{% endtabs %}

* Reboot for the configuration changes to take effect

```bash
sudo reboot
```

## System update

* Update the operating system and all installed software packages

```sh
sudo apt update && sudo apt full-upgrade
```

{% hint style="info" %}
Do this regularly for security-related updates
{% endhint %}

## Install general dependency packages

* Make sure that all necessary software packages are installed. Press "**y**" and `enter` or directly `enter` when the prompt asks you

```bash
sudo apt install git
```

## Check drive performance

Performant unit storage is essential for your node. Let's check if your drive works well as-is, or if additional configuration is needed.

* Install the software to measure the performance of your drive. Press "**y**" and `enter` or directly `enter` when the prompt asks you

```bash
sudo apt install hdparm
```

* Your external disk should be detected as `/dev/sda` if you use SSD or `/dev/nvme0n1` if you use NVMe. Check if this is the case by listing the names of connected block devices

```sh
lsblk -pli
```

#### Measure the speed of your drive depending on your case:

{% tabs %}
{% tab title="Case USB SSD (more common)" %}
Measure the speed of your drive

```bash
sudo hdparm -t --direct /dev/sda
```

**Example** of expected output:

```
Timing O_DIRECT disk reads: 932 MB in 3.00 seconds = 310.23 MB/sec
```

{% hint style="info" %}
If the speed of your USB3 drive is not acceptable, we need to configure the USB driver to ignore the UAS interface.

Check the [Fix bad USB3 performance](../troubleshooting.md#fix-bad-usb3-performance) entry in the [Troubleshooting](../troubleshooting.md) guide to learn how.
{% endhint %}
{% endtab %}

{% tab title="Case NVMe" %}
Measure the speed of your drive

```bash
sudo hdparm -t --direct /dev/nvme0n1
```

**Example** of expected output:

```
Timing O_DIRECT disk reads: 2422 MB in 3.00 seconds = 806.86 MB/sec
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
If the measured speeds are more than 150 MB/s, you're good but it is recommended more for a better experience
{% endhint %}

## Data directory

We'll store all application data in the dedicated directory `/data`. This allows for better security because it's not inside any user's home directory. Additionally, it's easier to move that directory somewhere else, for instance to a separate drive, as you can just mount any storage option to `/data`

* Create the data folder

```sh
sudo mkdir /data
```

* Assing to the `admin` user as the owner of the **`/data`** folder

```sh
sudo chown admin:admin /data
```

## Increase swap file size <a href="#increase-swap-file-size" id="increase-swap-file-size"></a>

The swap file acts as slower memory and is essential for system stability. The standard size of 200M is way too small.

* Edit the configuration file and comment the entry `CONF_SWAPSIZE` by placing a `#` in front of it. Save and exit

<pre class="language-bash"><code class="lang-bash"><strong>sudo nano /etc/dphys-swapfile
</strong></code></pre>

```
# comment or delete the CONF_SWAPSIZE line. It will then be created dynamically
#CONF_SWAPSIZE=200
```

* Recreate new swapfile

```bash
sudo systemctl restart dphys-swapfile
```
