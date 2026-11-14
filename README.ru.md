# OpenWrt для Nokia G-1425-MA и G-1426-MA

OpenWrt (snapshot, Linux 6.18) для двух GPON-терминалов Nokia. Это **разное железо**
(другой процессор, порядок байт, Wi-Fi и раскладка флеша), поэтому у каждой модели своя
прошивка. **Никогда не шейте образ одной модели на другую.** Заводская прошивка остаётся
в другом банке флеша как страховка.

English: [README.md](README.md) — там же раскладка флеша и подробности сборки.
Исходники: [zortald/gpon_openwrt](https://github.com/zortald/gpon_openwrt) (форк OpenWrt, конфиги сборки в `configs/`).
[Дампы железа](hardware/) с проверенных ONU.

## Какая у меня модель

| | G-1425-MA | G-1426-MA |
|---|---|---|
| Процессор | EcoNet EN7526GT | EcoNet EN7528DU |
| `/usr/hwconfig/` в стоке | — | `G1426MA/` (`BoardInnerID=G1426MA`) |
| Загрузчик в консоли | `EN751221 ... free bootbase` | `EN7528 ... free bootbase` |
| Wi-Fi в стоке | MT7603 + MT7662 (802.11ac) | MT7915 (Wi-Fi 6) |

Другие ревизии и прошивки могут отличаться — проверьте перед прошивкой.

## Скачать

| | G-1425-MA | G-1426-MA |
|---|---|---|
| Релиз | [v40](https://github.com/zortald/gpon_openwrt_packages/releases/tag/v40) | [g1426ma-v8](https://github.com/zortald/gpon_openwrt_packages/releases/tag/g1426ma-v8) |
| Установка со стока | [g1425-openwrt-v40-web-upgrade.bin](https://github.com/zortald/gpon_openwrt_packages/releases/download/v40/g1425-openwrt-v40-web-upgrade.bin) | [g1426-openwrt-v8-web-upgrade.bin](https://github.com/zortald/gpon_openwrt_packages/releases/download/g1426ma-v8/g1426-openwrt-v8-web-upgrade.bin) |
| Предварительный релиз (на железе ещё не проверен) | [v41](https://github.com/zortald/gpon_openwrt_packages/releases/tag/v41): калибровка 5 ГГц из стока, usteer, первый `sysupgrade` | — |
| Обновление из OpenWrt | — (переустановка со стока; `sysupgrade` начиная с v41) | [g1426-openwrt-v8-sysupgrade.bin](https://github.com/zortald/gpon_openwrt_packages/releases/download/g1426ma-v8/g1426-openwrt-v8-sysupgrade.bin) (`sysupgrade -n`) |

## Железо

Каждая колонка проверена на одной ONU.

| | G-1425-MA | G-1426-MA |
|---|---|---|
| Стоковая прошивка | `3FE46343AFIA94` | `3FE49218HJIJ62` (HJI.J62p05) |
| Процессор | EN7526GT, MIPS 34Kc, 1 ядро, big-endian | EN7528DU, MIPS 1004Kc, 2 ядра × 2 потока, 900 МГц, little-endian |
| Память / флеш | 512 МБ DDR3 / 256 МБ SPI NAND (Micron MT29F2G01) | 512 МБ DDR3 / 256 МБ SPI NAND (Micron MT29F2G01) |
| Ethernet | 4 × 1 GbE (MT7530) | 4 × 1 GbE (встроенный свитч EN7528) |
| Wi-Fi | MT7603E 2,4 ГГц + MT7662E 5 ГГц, 802.11ac | MT7915 2,4 + 5 ГГц, 802.11ax |
| USB | 1 × USB 3.0 | 1 × USB 3.0 |
| Консоль | J521 (GND/TX/RX), 3,3 В, 115200 8N1 | 3,3 В, 115200 8N1 |
| Не поддерживается | GPON | GPON, телефонный порт |
| Дампы | [hardware/](hardware/) | [hardware/g1426ma/](hardware/g1426ma/) |

## Установка

1. Выньте кабель провайдера из порта 1 (у стока все порты — LAN с DHCP).
2. Откройте `http://192.168.1.1/upgrade.cgi` (пункт меню скрыт, страница работает).
   G-1426-MA: сначала войдите (`telecomadmin` / `admintelecom`).
3. Выберите файл **для своей модели**, нажмите Upgrade. Сток запишет другой банк,
   переключит флаг загрузки и перезагрузится в OpenWrt.
4. Только G-1425-MA: если после первой загрузки нет сети — один раз перезагрузите по питанию.

Загрузчик и разделы стока не меняются.

| | G-1425-MA | G-1426-MA |
|---|---|---|
| Сток работает из | банка A | банка B |
| OpenWrt ставится в | банк B | банк A |

## По умолчанию

- LAN `10.100.0.1/24` с DHCP, порты 2–4; порт 1 — WAN (DHCP-клиент)
- Wi-Fi открытый, 2,4 + 5 ГГц: SSID `G1425` / `G1426`
- LuCI: `http://10.100.0.1` — сразу задайте пароль root
- G-1426-MA: аппаратный NAT (PPE) для проводных LAN↔WAN, ~900 Мбит/с без нагрузки на процессор
- Часовой пояс Europe/Kyiv; USB-модемы, USB-накопители, WireGuard
- Настройки и пакеты — в томе 1 бывшего стокового `/data` (~130 МБ / ~112 МБ),
  том 0 оставлен стоку; при переключении на сток и обратно настройки сохраняются
- Модули ядра ставятся из этого репозитория (`apk`), фид подключается сам

## Фид пакетов

| Папка | Модель | Архитектура | Прошивка |
|---|---|---|---|
| [af8c55194775dcae215a7df4c97d3160](af8c55194775dcae215a7df4c97d3160/) | G-1425-MA | `mips_24kc` | v38–v40 |
| [7d991811584158991fd8b3ea9fb13c0b](7d991811584158991fd8b3ea9fb13c0b/) | G-1425-MA | `mips_24kc` | v41 |
| [6f45af18e396b5d378e1a1baaebc5a16](6f45af18e396b5d378e1a1baaebc5a16/) | G-1426-MA | `mipsel_24kc` | g1426ma-v5, v6, v8 |

## Ограничения

| | G-1425-MA | G-1426-MA |
|---|---|---|
| GPON | не работает | не работает |
| Процессор | одно ядро | все четыре потока |
| Обновление | переустановка через веб стока; `sysupgrade` с v41 (предварительный) | `sysupgrade` |
| Прочее | — | телефонный порт не работает |

## Вернуться в сток

G-1425-MA, из OpenWrt:

    printf '\0\0\0\0\0\0\0\0\0\0\0\001\0\0\0\016' > /tmp/f; mtd write /tmp/f flag; reboot

G-1426-MA, из OpenWrt:

    en75_chboot factory; reboot

Если OpenWrt не загружается, загрузчик сам вернёт сток после исчерпания счётчика попыток.
