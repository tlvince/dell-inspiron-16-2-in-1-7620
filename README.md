# Inspiron 16 2-in-1 7620

Device notes and configuration under Linux for the [Inspiron 16 2-in-1 7620](https://www.dell.com/en-uk/shop/laptops/inspiron-16-2-in-1-laptop/spd/inspiron-16-7620-2-in-1-laptop/cn76201sc) 4K OLED variant.

## Specs

- [Intel i7-1260P](https://ark.intel.com/content/www/us/en/ark/products/226254/intel-core-i71260p-processor-18m-cache-up-to-4-70-ghz.html) (Alder Lake)
- NVIDIA MX550 (Turing)
- Samsung PM991a SSD
- Samsung SDC4164 4k OLED 3840x2400 400 nits
- 87Wh battery
- HDMI 1.4 only

- https://www.notebookcheck.net/Dell-Inspiron-16-7620-2-in-1-convertible-review-Mylar-and-aluminum-chassis.628030.0.html
- https://linux-hardware.org/?probe=7ad1831ce9

## Notes

- clone of MacBook Pro 16 design wise, marginally heavier, slightly thicker
- poor tinny speakers compared to MacBook
- Windows front firing speaker mixer bug
- feature-full BIOS
- NVMe set in RAID mode on the controller by default for all modern Dells regardless of by number of m2 slots
  - [Intel Rapid Storage Technology](https://en.m.wikipedia.org/wiki/Intel_Rapid_Storage_Technology)
  - aka [FakeRAID](https://wiki.archlinux.org/title/RAID#Implementation)
  - requires `vmd` kernel module
  - `vmd` can cause [high power usage](<https://wiki.archlinux.org/title/Dell_XPS_13_Plus_(9320)#Cannot_enter_S0ix_causing_high_power_usage>)
  - [switching from RAID to AHCI](https://gist.github.com/chenxiaolong/4beec93c464639a19ad82eeccc828c63#switching-from-raid-to-ahci)
- everything working out of the box in Linux (including touchscreen, rotation, fingerprint) except the front firing speakers (fixed in next kernel release)
- can set custom charge limit in BIOS
- can’t disable dGPU in BIOS
- Windows defaults to 2.5x scaling
- portability and tablet mode hampered by the size and weight
- touchpad mylar layer prevents capacitive grounding "vibration"
- both USB C ports are on the same side - minor annoyance depending on desk set up
- shame active stylus not included
- quieter fan than Framework
