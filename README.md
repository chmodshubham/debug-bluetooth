# Fixing Bluetooth Device Discovery Issues on Ubuntu (Realtek RTL8852BU)

This guide documents the steps to debug and fix the issue where **Bluetooth devices are not showing up in Ubuntu** after removing/forgetting them.  
Tested on systems with **Realtek RTL8852BU** Bluetooth chip, but should work for similar adapters.

## 1. Make Sure Your System is Fully Updated

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential git dkms linux-headers-$(uname -r)
````

## 2. Update Realtek Bluetooth Firmware

Ubuntu ships `/lib/firmware/rtl_bt/rtl8852bu_fw.bin` and `rtl8852bu_config.bin`, but sometimes they’re outdated.
Reinstall the latest firmware package:

```bash
sudo apt install --reinstall linux-firmware
```

## 3. Full Bluetooth Reset (Clear Cache & Restart Service)

```bash
sudo systemctl stop bluetooth
sudo rm -rf /var/lib/bluetooth/*
sudo systemctl start bluetooth
```

## 4. Reload the Bluetooth Kernel Driver

This forces the adapter to reinitialize and reload its firmware.

```bash
sudo systemctl stop bluetooth
sudo modprobe -r btusb
sudo modprobe btusb
sudo systemctl start bluetooth
```

## 5. Reload Systemd Service Files (If Bluetooth Unit Changed)

If you see a warning like:

```
Warning: The unit file, source configuration file or drop-ins of bluetooth.service changed on disk.
```

Run:

```bash
sudo systemctl daemon-reload
sudo systemctl restart bluetooth
```

Check service status:

```bash
systemctl status bluetooth
```

## 6. Pairing the Device Using `bluetoothctl`

1. Put your earphones/headset into **pairing mode**.
2. Run the following commands:

```bash
bluetoothctl
power on
agent on
default-agent
scan on
```

3. Once your device appears, note its MAC address and run:

```bash
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
```

Replace `XX:XX:XX:XX:XX:XX` with the actual address from the scan output.

## Notes

* These steps are particularly useful for **Realtek Bluetooth adapters** which sometimes fail to rediscover devices due to firmware or driver glitches.
* If the problem keeps coming back, consider installing the **latest Realtek driver** from source via DKMS.
