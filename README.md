# Inspiron 16 2-in-1 7620

Device notes and configuration under Linux for the [Inspiron 16 2-in-1 7620](https://www.dell.com/en-uk/shop/laptops/inspiron-16-2-in-1-laptop/spd/inspiron-16-7620-2-in-1-laptop/cn76201sc) 4K OLED variant.

Everything works of the box in Linux v6.0 (including touchscreen, rotation, fingerprint scanner, SD-card reader), with the exception of the front firing speakers (a [patch](https://github.com/tiwai/sound/commit/2912cdda734d9136615ed05636d9fcbca2a7a3c5) has been merged and is expected to ship in v6.2).

## Specs

- [Intel i7-1260P](https://ark.intel.com/content/www/us/en/ark/products/226254/intel-core-i71260p-processor-18m-cache-up-to-4-70-ghz.html) (Alder Lake)
- NVIDIA MX550 (Turing)
- Samsung PM991a M.2 2230 SSD (supports M.2 2280 PCIe Gen4)
- Samsung SDC4164 4k OLED 3840x2400 400 nits 283ppi
- 87Wh battery
- 2x SODIMM (supports up to 64GB, DDR4)
- 1x M.2 NVMe
- HDMI 1.4 only
- [Hardware for Linux probe](https://linux-hardware.org/?probe=3e34521c45)

## Opinion

- [Notebookcheck review](https://www.notebookcheck.net/Dell-Inspiron-16-7620-2-in-1-convertible-review-Mylar-and-aluminum-chassis.628030.0.html)
- reminiscent of MacBook Pro 16 design wise, marginally heavier, slightly thicker
- acceptable speaker volume, poor bass response
- portability and tablet mode hampered by the size and weight
- touchpad mylar layer prevents capacitive grounding "vibration"
- both USB C ports are on the same side - minor annoyance depending on desk set up
- no stylus included (shame)
- anecdotally quieter fan than Framework
- excellent OLED display with 283ppi - [ideal for HiDPI](https://github.com/cassidyjames/dippi/blob/0703424c3d541f581bd83d519da365cfd8566dda/dpi.md), running at 2x (integer) scaling
  - unfortunately this unit has grey uniformity issues
  - only visible in a dark room with dark grey backgrounds, e.g. hex [#111111](http://color.aurlien.net/#111111) (coincidentally, the default for the [foot terminal](https://codeberg.org/dnkl/foot)) and [#0f0f0f](http://color.aurlien.net/#0f0f0f)
  - common downside for OLED
  - best to use a pure black background for power saving

## BIOS

- GUI-based UEFI
- _can_ set custom charge limit
- can’t disable dGPU
- partial Linux support via `libsmbios`
- NVMe RAID mode set by default for all modern Dells regardless of number of M.2 slots
  - [Intel Rapid Storage Technology](https://en.m.wikipedia.org/wiki/Intel_Rapid_Storage_Technology)
  - aka [FakeRAID](https://wiki.archlinux.org/title/RAID#Implementation)
  - requires `vmd` kernel module
  - `vmd` can cause [high power usage](<https://wiki.archlinux.org/title/Dell_XPS_13_Plus_(9320)#Cannot_enter_S0ix_causing_high_power_usage>)
  - [switching from RAID to AHCI](https://gist.github.com/chenxiaolong/4beec93c464639a19ad82eeccc828c63#switching-from-raid-to-ahci)
- no "legacy" S3 deep sleep option

## Software

- audio requires `sof-firmware`
- [getUserMedia / getDisplayMedia Test Page](https://mozilla.github.io/webrtc-landing/gum_test.html) - useful for testing webcam/screensharing
- av1 decoding broken in [Firefox v107](https://bugzilla.mozilla.org/show_bug.cgi?id=1793507)
  - fixed in v108, workaround: `media.ffvpx.enabled: false`

## Sleep

- S3 sleep unsupported
  - S0ix supported, reaches S0i2.0
- consumed ~4.7% battery in ~12 hours

## NVIDIA

- Full [RTD3 Power Management](https://download.nvidia.com/XFree86/Linux-x86_64/525.60.11/README/dynamicpowermanagement.html) supported
  - [RTD3 configuration](<https://wiki.archlinux.org/title/PRIME#PCI-Express_Runtime_D3_(RTD3)_Power_Management>)
- CUDA supported

Test environment:

- idle
- 30% brightness
- sway
- Firefox open
- WiFi connected
- Bluetooth enabled
- tlp, thermald daemons running (default configs)

Verify [PCI state](https://docs.kernel.org/power/pci.html#native-pci-power-management) via:

```sh
cat /sys/class/drm/card1/device/power_state
```

| Driver        | Power Consumption | PCI state |
| ------------- | ----------------- | --------- |
| nvidia        | 6.7W              | D3cold    |
| nvidia-open   | 10.8W             | D0        |
| nouveau       | 6.7W              | D3cold    |
| (blacklisted) | 6.73W             | -         |

### nvidia

- [requires `ibt=off`](https://bugs.archlinux.org/task/74886) to boot
- `sway --unsupported-gpu`

### nvidia-open

- PCI utilisation: 100%
  - [power management unimplemented](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/314)
- ~7W in vconsole
  - ~3W for NVIDIA alone

### blacklisted

- using [uDev rules](https://wiki.archlinux.org/title/Hybrid_graphics#Using_udev_rules)
- not in the PCI tree

## Near-idle benchmarks

Testing near-idle performance via two open foot terminals, one running `powerstat` with the following flags:

```sh
powerstat -d 0 -c -H 1 480
```

... the other running a bash one-liner:

```sh
while [ 1 ]; echo 'foo'; do sleep 1; done
```

Test environment:

- idle
- Wayland only (XWayland disabled)
- 40% brightness
- WiFi connected
- tlp, thermald daemons running (default configs)

Comparing GNOME and sway with roughly equivalent helpers to provide a basic "desktop environment".

### sway (no bar)

- sway without a bar (`swaybar` not running)
- `swayidle`
- `swaybg`
- `gammastep`

| State   | C10%   | Power (W) |
| ------- | ------ | --------- |
| sleep 1 | 99.63% | 6.08      |

### sway (waybar)

- sway
- `waybar` with modules:
  - sway/workspaces
  - sway/mode
  - sway/window
  - network
  - battery
  - tray
  - clock#date
- `swayidle`
- `swaybg`
- `gammastep`

| State   | C10%   | Power (W) |
| ------- | ------ | --------- |
| sleep 1 | 98.32% | 6.81      |

### GNOME

- `gnome-shell --no-x11`
- `gdm3`
- `gnome-settings-daemon`

| State      | C10%   | Power (W) |
| ---------- | ------ | --------- |
| idle       | 99.69% | 5.37      |
| sleep 1    | 99.56% | 5.48      |
| sleep 0.1  | 98.64% | 6.51      |
| sleep 0.01 | 90.6%  | 9.36      |

## Links

- [lakotamm/Dell-Inspiron-16-Plus-7620-Linux-Config](https://github.com/lakotamm/Dell-Inspiron-16-Plus-7620-Linux-Config)
  - [lakotamm /r/dell discussion](https://old.reddit.com/r/Dell/comments/w30svq/freshly_new_inspiron_16_plus_7620_with_i7_and_rtx/)
- [lhl/linuxlaptops - Framework Laptop](https://github.com/lhl/linuxlaptops/wiki/2022-Framework-Laptop-DIY-Edition-12th-Gen-Intel-Batch-1)
- [Dell Inspiron 16 2-in-1 (7620) - ArchWiki](https://wiki.archlinux.org/title/Dell_Inspiron_16_2-in-1_(7620))
- [Service Manual](https://dl.dell.com/content/manual16923540-inspiron-16-7620-2-in-1-service-manual.pdf)
- [Setup and Specifications](https://dl.dell.com/content/manual17399228-inspiron-16-7620-2-in-1-setup-and-specifications.pdf)
