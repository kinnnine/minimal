---
state:      post
title:      Madlion--MAD60HE Quick & Dirty HID Reverse-engineering.
date:       2026-05-11 13:02:13 +07:00
---

HID data are captured via Wireshark while using the Madlion Web Driver, alongside with devtool console in the new tab page on Linux machine.

Madlion Web Driver: [https://hub.fgg.com.cn/](https://hub.fgg.com.cn/)

## Device Information

```
ID 373b:1053 Shenzhen Yizhita Technology Co., Ltd MAD60
```

## Setup Snippets

```
sudo modprobe usbmon
sudo setfacl -m u:$USER:r /dev/usbmon*
```

## Wireshark Snippets

```
BUS.DEVICE.ENDPOINT (look up lsusb command)
1  .xx    .0

HID Data usually lives on endpoint 3.
"HID Data" also shown as "Leftover Capture Data" in Wireshark, but they are the same.

Host: our pc
Keyboard: the device itself

Descriptior:
usb.addr == "1.xx.0"

Host > Keyboard:
usb.addr == "1.xx.2" && usb.urb_type == URB_SUBMIT
usb.addr == "1.xx.3" && usb.urb_type == URB_SUBMIT

Host < Keyboatd:
usb.addr == "1.xx.2" && usb.urb_type == URB_COMPLETE
usb.addr == "1.xx.3" && usb.urb_type == URB_COMPLETE
```

## JavaScript Snippets

```
device[x] should have usage page: 97
sendReport's report id is always 0x00.

Initialize:
const device = await navigator.hid.requestDevice({ filters: [] });
await device[x].open

Construct:
const hex = "hid_hex_command"
const data = Uint8Array.fromHex(hex)

Send:
await device[x].sendReport(0x00, data)
```

## HID Hex Commands

#### Key assignment.

```
A=command, B=key_layer, C=key_row, D=key_usage_id, .=reserved,unused:
0500000000000000000000000000000000000000000000000000000000000000
AABB.CCCDDDD....................................................

05<key_layer>0<key_row><key_usage_id>

Key Rows: 000-40D
Key Layers: 00-03

Key Layer 00: Normal Layer
Key Layer 01: FN1
Key Layer 02: FN2 (Mac)
Key Layer 03: FN3
```

```
Special key code usage ids:

Penetrate Keystroke: 0029

Macro Key: 7700-770F

Win Fn: 5221
Mac Fn: 5223

Mouse Move Up: 00CD
Mouse Move Down: 00CE
Mouse Move Left: 00CF
Mouse Move Right: 000D

Mouse Left Click: 00D1
Mouse Right Click: 00D2
Mouse Middle Click: 00D3

Mouse Back Click: 00D4
Mouse Forward Click: 00D5

Mouse Scroll Up: 00D9
Mouse Scroll Down: 00DA
Mouse Scroll Left: 00DB
Mouse Scroll Right: 00DC

MacOS Scheduling Center: 00C1
MacOS Launchpad: 00C2

Toggle Swap WASD: 7E01
Win/Mac Layout Swap: 7E02
Toggle Win Lock: 7E03
Switch to 6-Key Rollover: 7E04
```

```
Light effect key usage ids also available here but useless on non-rgb model:

Toggle RGB Light: 7820

Next RGB Mode: 7821
Previous RGB Mode: 7822

Increase Color Hue: 7823
Decrease Color Hue: 7824

Increase Brightness: 7827
Decrease Brightness: 7828

Increase RGB effect speed: 7829
Decrease RGB effect speed: 782A
```

#### Swap WASD, Mac Mode, Win lock and 6-key rollover.

```
All of these 4 parameters are under the same command.
Swap WASD, Mac Mode, Win lock, 6-key rollover

In the example below is the default configuration (all turned off).
A=command, B=swap_wasd, C=mac_mode, D=win_lock, E=six_key_rollover, .=reserved,unused:
0396110000000001000000000000000000000000000000000000000000000000
AAAAAA..BBCCDDEE................................................

03961100<swap_wasd><mac_mode><win_lock><six_key_rollover>

Parameters for all except 6-key rollover:
01 = On
00 = Off

Only 6-key rollover parameter is swapped, like below:
00 = On
01 = Off

Still not understand?
Maybe more example below should help.

All turned on:
0396110001010100000000000000000000000000000000000000000000000000
AAAAAA..BBCCDDEE................................................
All turned off (default):
0396110000000001000000000000000000000000000000000000000000000000
AAAAAA..BBCCDDEE................................................
Only 6-key rollover turned on:
0396110000000000000000000000000000000000000000000000000000000000
AAAAAA..BBCCDDEE................................................
```

#### Touch Prevention Mode

```
A=command, B=parameter, .=reserved,unused:
03961d0100000000000000000000000000000000000000000000000000000000
AAAAAABB........................................................

Parameters:
01 = On
00 = Off
```

# References

[https://usb.org/sites/default/files/hut1_3_0.pdf#page=89](https://usb.org/sites/default/files/hut1_3_0.pdf#page=89)

[https://wiki.wireshark.org/CaptureSetup/USB](https://wiki.wireshark.org/CaptureSetup/USB)
