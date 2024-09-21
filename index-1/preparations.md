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

# 1.1 Preparations

Let's get all the necessary hardware parts and prepare some passwords.

<figure><img src="../.gitbook/assets/Starting_MiniBolt.gif" alt=""><figcaption></figcaption></figure>

## Raspberry Pi

This guide builds on the readily available Raspberry Pi.

While a personal computer is the best choice, this guide works with other computing platforms, cloud servers, or virtual machines.

<div>

<figure><img src="../.gitbook/assets/raspberry-pi-5-model-b.jpg" alt="" width="375"><figcaption><p>Raspberry Pi 5B</p></figcaption></figure>

 

<figure><img src="../.gitbook/assets/raspberry-pi-m-2-hat.jpg" alt="" width="375"><figcaption><p>Raspberry Pi M.2 HAT+ (optional)</p></figcaption></figure>

</div>

## Hardware requirements

You need the following hardware:

* Raspberry Pi 4B/5B (recommended) - with 4GB/8GB (recommended) RAM
* Official Raspberry Pi power adapter: other adapters are a common cause of reliability issues
* Storage:
  * Option 1: External storage: SSD with USB3 (2+ TB is recommended)
  * Option 2 (only Raspberry Pi 5): Internal storage: Raspberry Pi M.2 HAT + M.2 2230 or 2242 form factor NVMe SSD (2+ TB is recommended) or with compatible shield 2260 or 2282 form factor
* A regular computer (laptop, PC, etc)

You might also want to get this optional hardware:

* UPS (uninterruptible power supply)
* Raspberry Pi case to protect your Pi and cool it down
* A small USB thumbdrive or microSD card to create regular local backups of your Lightning channels

## Write down your passwords

You will need several passwords, and it's easiest to write them all down in the beginning, instead of bumping into them throughout the guide. They should be unique and very secure, at least 12 characters in length. Do **not use uncommon special characters**, spaces, or quotes (‘ or “).

```
[ A ] Master admin user password
[ B ] Bitcoin RPC password
[ C ] LND wallet password
[ D ] BTC-RPC-Explorer password (optional)
[ E ] ThunderHub password
```

![](../images/preparations\_xkcd.png)

If you need inspiration for creating your passwords: the [xkcd: Password Strength](https://xkcd.com/936/) comic is funny and contains a lot of truth. Store a copy of your passwords somewhere safe (preferably in an open-source password manager like [KeePassXC](https://keepassxc.org/)), or whatever password manager you're already using, and keep your original notes out of sight once your system is up and running.

## Secure your home network and devices

While the guide will show you how to secure your node, you will interact with it from your computer and mobile phone and use your home internet network. Before building your MiniBolt, it is recommended to secure your home network and devices. Follow Parts 1 and 2 of this ["How to Secure Your Home Network Against Threats"](https://restoreprivacy.com/secure-home-network/) tutorial by Heinrich Long, and try to implement as many points as possible (some might not apply to your router/device).
