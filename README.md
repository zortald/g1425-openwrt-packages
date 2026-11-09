# OpenWrt for Nokia G-1425-MA (EcoNet EN7526GT)

OpenWrt (snapshot, Linux 6.18) for the **Nokia G-1425-MA** GPON ONT as shipped by
**Westelecom** (Ukraine). It runs from the second firmware bank; the stock firmware
stays in the first bank as a fallback.

This repository holds:

- the firmware image for the stock web upgrade — see [Releases](https://github.com/zortald/g1425-openwrt-packages/releases);
- the **package feed** with kernel modules and target packages for this build
  (served at `https://zortald.github.io/g1425-openwrt-packages/`);
- [hardware dumps](hardware/) from the unit the firmware was verified on.

Русская версия: [README.ru.md](README.ru.md)

## Download

Latest: **[g1425-openwrt-v40-web-upgrade.bin](https://github.com/zortald/g1425-openwrt-packages/releases/download/v40/g1425-openwrt-v40-web-upgrade.bin)**
([release notes and checksums](https://github.com/zortald/g1425-openwrt-packages/releases/tag/v40))

## Supported hardware

Verified on one unit with this hardware. Other G-1425 revisions or other ISP
firmwares may use a different SoC, flash layout or bootloader — check before flashing.

| | |
|---|---|
| Device | Nokia G-1425-MA |
| Stock firmware | `3FE46343AFIA94` (Westelecom), Linux 3.18.21 |
| SoC | EcoNet **EN7526GT** (EN751221 family), MIPS 34Kc, 1 core, big-endian, 597 BogoMIPS |
| RAM | 512 MB DDR3, ~427 MB usable |
| Flash | 256 MB SPI NAND, Micron MT29F2G01 (2048 blocks, 163 reserved for BMT, 235 MB usable) |
| Bootloader | EcoNet "EN751221 free bootbase" v1.1 (2021), dual bank with a boot flag |
| Ethernet | 4 × 1 GbE; on-die MT7530 in passthrough + multi-chip-module MT7530 with GE PHYs `03a2:9412` |
| Wi-Fi 2.4 GHz | MediaTek MT7603E (PCIe `14c3:7603`), driver mt7603e |
| Wi-Fi 5 GHz | MediaTek MT7662E 2×2 802.11ac (PCIe `14c3:7662`), driver mt76x2e |
| USB | 1 × USB 3.0 (xHCI) |
| Serial | J521, 3 pins (GND/TX/RX, no VCC), 3.3 V, 115200 8N1 |
| GPON | EN7526GT xPON MAC + GN25L98 laser driver — **not supported** under OpenWrt |

### How to tell you have the same hardware

- Stock web UI shows **GPON Gateway, Type: HGU** and the stock firmware version above.
- The chip under the heatsink is marked **ECONET EN7526GT**.
- Over serial, the bootloader prints `EN751221 ... free bootbase` and
  `Output Enable Gpio 6 for 7526G!` (see [hardware/bootloader.txt](hardware/bootloader.txt)).

Full logs from the verified unit (MAC addresses redacted):

- [hardware/bootloader.txt](hardware/bootloader.txt) — bootloader output
- [hardware/dmesg.txt](hardware/dmesg.txt) — kernel log: CPU, NAND/BMT, PCIe, Wi-Fi, switch, USB
- [hardware/system.txt](hardware/system.txt) — `/proc/cpuinfo`, memory, `/proc/mtd`, release

## Install (stock web upgrade)

1. **Unplug the cable from port 1.** In stock firmware all four ports are LAN with a DHCP server.
2. Connect a computer to a LAN port and open `http://192.168.1.1/upgrade.cgi`
   (the menu item is hidden for regular users, the page itself works).
3. Choose the `.bin` file and press Upgrade. The stock updater writes the second bank,
   switches the boot flag and reboots.
4. If there is no network after the very first boot, power-cycle the ONU once.

The stock updater checks only the image header and CRC. The image leaves the bootloader
and the stock partitions untouched.

## Defaults

| | |
|---|---|
| LAN | `10.100.0.1/24`, DHCP server, ports 2–4 |
| WAN | port 1, DHCP client |
| MAC | WAN = factory MAC, LAN = factory MAC − 1, Wi-Fi = +1 / +2 |
| Wi-Fi | **open** network `G1425`, 2.4 + 5 GHz, 802.11k/v, country UA |
| Web UI | LuCI at `http://10.100.0.1` — **set a root password first** |
| Time zone | Europe/Kyiv |
| Extras | ModemManager + usb-modeswitch (USB modems), USB storage (ext4/vfat), WireGuard |

## Flash layout

| mtd | Name | Size | Used by |
|---|---|---|---|
| 0 | bootloader | 256 KB | read-only |
| 1 | romfile | 256 KB | read-only |
| 2 | stock_kernel | 3 MB | stock, bank A (read-only) |
| 3 | stock_rootfs | 20 MB | stock, bank A (read-only) |
| 4 | kernel_slave | 3 MB | OpenWrt kernel, bank B |
| 5 | rootfs_slave | 20 MB | OpenWrt squashfs (UBI) |
| 6 | bosa | 256 KB | optics calibration (read-only) |
| 7 | ri | 256 KB | factory data, MAC at 0x3e (read-only) |
| 8 / 9 | flag / flagback | 256 KB each | boot bank selection |
| 10 | stock_config | 10 MB | stock settings incl. GPON (read-only) |
| 11 | openwrt_data | 160 MB | former stock `/data`: volume 0 `data` (8 MB, for stock), volume 1 `rootfs_data` (OpenWrt settings and packages, ~130 MB) |
| 12 | stock_log | 10 MB | stock logs (read-only) |

Stock mounts volume 0 of mtd11 as its `/data` and would reformat the whole partition if
that failed, so OpenWrt keeps its overlay in volume 1. Settings survive switching to
stock and back, and survive reflashing.

The boot flag holds four big-endian words: active bank, current image, boot-confirmed,
attempt counter. The bootloader counts down unconfirmed boots and falls back to bank A
at zero; OpenWrt confirms a good boot (`/etc/init.d/bootconfirm`).

## Package feed

`downloads.openwrt.org` does not build the econet target, so its kernel modules and
target libraries are not available there. This feed provides them.

Kernel modules only install on the exact kernel they were built with, so there is one
folder per kernel build, named by the kernel package hash. The firmware adds its folder
to apk at first boot:

    https://zortald.github.io/g1425-openwrt-packages/<kernel-hash>/packages.adb

Userspace packages come from the official `mips_24kc` snapshot feeds. Packages here are
signed with the build key shipped in the firmware.

| Folder | Kernel | Firmware |
|---|---|---|
| [af8c55194775dcae215a7df4c97d3160](af8c55194775dcae215a7df4c97d3160/) | 6.18.60~af8c5519…-r1 | v38–v40 |

## Back to stock

From OpenWrt:

    printf '\0\0\0\0\0\0\0\0\0\0\0\001\0\0\0\016' > /tmp/f; mtd write /tmp/f flag; reboot

If OpenWrt does not boot at all, the bootloader returns to stock by itself after the
unconfirmed-boot counter runs out. Over serial, erasing the flag blocks in the bootloader
(`nander 2f00000 20000 0`, `nander 2f40000 20000 0`, `reset`) also boots stock; the
bootloader asks for the vendor login first.

## Known limitations

- No GPON: the ONT does not work as a fiber terminal under OpenWrt.
- Only one CPU core is used.
- No `sysupgrade` yet: new versions are installed from stock via the web upgrade.
- Snapshot build: userspace packages from the official feed may occasionally need newer
  libraries than the firmware has.

## Build

OpenWrt snapshot (`econet/en751221`, kernel 6.18.60) with
[openwrt/openwrt#25330](https://github.com/openwrt/openwrt/pull/25330) (EcoNet DSA switch
support) and a device definition for the G-1425-MA: device tree, partition map,
HDR3 image header for the stock bootloader and updater.
