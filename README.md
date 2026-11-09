# OpenWrt package feed for Nokia G-1425-MA (EcoNet EN7526GT / EN751221)

Kernel modules and target packages for the custom OpenWrt build that runs on the
Nokia G-1425-MA ONT. downloads.openwrt.org does not build the econet target, so
these packages are not available there.

Kernel modules only install on the exact kernel they were built with, so there is
one folder per kernel build, named by the kernel package hash. Each firmware image
adds its own folder to apk at first boot:

    https://zortald.github.io/g1425-openwrt-packages/<kernel-hash>/packages.adb

Packages are signed with the build key shipped in the firmware.
Userspace packages come from the official mips_24kc snapshot feeds.

| Folder | Kernel |
|---|---|
| [af8c55194775dcae215a7df4c97d3160](af8c55194775dcae215a7df4c97d3160/) | 6.18.60~af8c55194775dcae215a7df4c97d3160-r1 |
