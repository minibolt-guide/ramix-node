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

You are now on the command line of your own Bitcoin node. Let's start with the configuration.

<figure><img src="../.gitbook/assets/configuration.jpg" alt="" width="375"><figcaption></figcaption></figure>

## System update

* Update the operating system and all installed software packages

```sh
sudo apt update && sudo apt full-upgrade
```

{% hint style="info" %}
Do this regularly every few months for security-related updates
{% endhint %}

## Check drive performance

Performant unit storage is essential for your node.

Let's check if your drive works well as-is.

* Your disk should be detected as `/dev/sda`. Check if this is the case by listing the names of connected block devices

```sh
lsblk -pli
```

* Measure the speed of your drive

```sh
sudo hdparm -t --direct /dev/sda
```

**Example** of expected output:

```
> Timing O_DIRECT disk reads: 932 MB in 3.00 seconds = 310.23 MB/sec
```

{% hint style="success" %}
If the measured speeds are more than 150 MB/s, you're good. If the measured speeds are more than 100 MB/s, you're good but is recommended more for a better experience
{% endhint %}

## Data directory

We'll store all application data in the dedicated directory `/data`. This allows for better security because it's not inside any user's home directory. Additionally, it's easier to move that directory somewhere else, for instance to a separate drive, as you can just mount any storage option to `/data`

* Create the data folder

```sh
sudo mkdir /data
```

* Assing to the `admin` user as the owner of the **`(/data)`** folder

```sh
sudo chown admin:admin /data
```
