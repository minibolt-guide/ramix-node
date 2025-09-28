# 🔧 Troubleshooting

## Fix bad USB3 performance <a href="#fix-bad-usb3-performance" id="fix-bad-usb3-performance"></a>

If the speed of your tested USB3 drive speed with `hdparm` in the [Check drive performance](index-1/configuration.md#check-drive-performance) section is not acceptable, we need to configure the USB driver to ignore the UAS interface.

* With user `admin`, get the `Vendor` and `Product ID` for your USB3 drive. Run the following command and look for the name of your drive or adapter. The relevant data is printed as `idVendor:idProduct` (`0bda:9210` in this example). Make a note of these values.

```bash
lsusb
```

**Example** of expected output:

```
Bus 002 Device 002: ID 0bda:9210 Realtek Semiconductor Corp. RTL9210 M.2 NVME Adapter
[...]
```

{% hint style="info" %}
The additional configuration parameters (called “quirks”) for the USB driver must be passed to the Linux kernel during the boot process
{% endhint %}

* Open the bootloader configuration file

```bash
sudo nano /boot/firmware/cmdline.txt
```

* **At the start of the line of parameters**, add the text `usb-storage.quirks=aaaa:bbbb:u` where `aaaa:bbbb` are the values you noted in the `lsusb` command above. Make sure that there is **a single space character** ( ) between our addition and the next parameter. Save and exit.

```
usb-storage.quirks=0bda:9210:u ..............
```

{% hint style="info" %}
If you have multiple drives that need these “quirks”, add them all to the single directive, separated by commas

```
usb-storage.quirks=0bda:9210:u,152d:0578:u ..............
```
{% endhint %}

* Reboot your node

```bash
sudo reboot
```

* Log in again as `admin` and test the USB3 drive performance again

```bash
sudo hdparm -t --direct /dev/sda
```

{% hint style="info" %}
You should see a significant increase in performance (+200MB/s). If the test still shows a very slow read speed, your drive or USB3 adapter might not be compatible with the Raspberry Pi. In that case, we recommend visiting the [Raspberry Pi Troubleshooting forum](https://forums.raspberrypi.com/viewforum.php?f=28) or simply trying out hardware alternatives
{% endhint %}

{% hint style="info" %}
_more:_ [_Raspberry Pi forum: bad performance with USB3 SSDs_](https://forums.raspberrypi.com/viewtopic.php?f=28\&t=245931)
{% endhint %}
