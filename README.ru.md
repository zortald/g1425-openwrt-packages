# OpenWrt для Nokia G-1425-MA (EcoNet EN7526GT)

OpenWrt (snapshot, Linux 6.18) для GPON-терминала **Nokia G-1425-MA**.
Работает из второго банка прошивки, заводская остаётся в первом как страховка.

English: [README.md](README.md) — там же подробная таблица железа, раскладка флеша и
[дампы с проверенной ONU](hardware/).

## Скачать

**[g1425-openwrt-v40-web-upgrade.bin](https://github.com/zortald/g1425-openwrt-packages/releases/download/v40/g1425-openwrt-v40-web-upgrade.bin)**
([описание и контрольные суммы](https://github.com/zortald/g1425-openwrt-packages/releases/tag/v40))

## Железо

Проверено на ONU с процессором **EcoNet EN7526GT** (семейство EN751221, MIPS 34Kc),
512 МБ DDR3, 256 МБ SPI NAND (Micron MT29F2G01), Wi-Fi MT7603E (2.4 ГГц) + MT7662E (5 ГГц ac),
4 × 1 GbE, USB 3.0, стоковая прошивка `3FE46343AFIA94`.
Признаки: в вебе стока «GPON Gateway, Type: HGU», на чипе маркировка ECONET EN7526GT,
загрузчик в консоли пишет `EN751221 ... free bootbase`.

## Установка

1. Выньте кабель из порта 1 (у стока все порты — LAN с DHCP).
2. Откройте `http://192.168.1.1/upgrade.cgi` (пункт меню скрыт, страница работает).
3. Выберите файл, нажмите Upgrade. ONU запишет второй банк и перезагрузится.
4. Если после первой загрузки нет сети — один раз перезагрузите по питанию.

Загрузчик и разделы стока не меняются.

## По умолчанию

- LAN `10.100.0.1/24` с DHCP, порты 2–4; порт 1 — WAN (DHCP-клиент)
- Wi-Fi открытый, SSID `G1425`, 2.4 + 5 ГГц
- LuCI: `http://10.100.0.1` — сразу задайте пароль root
- USB-модемы, USB-накопители, WireGuard; модули ядра ставятся из этого репозитория

## Ограничения

- Оптика (GPON) под OpenWrt не работает.
- Обновление — только через веб стока (sysupgrade пока нет).

## Вернуться в сток

    printf '\0\0\0\0\0\0\0\0\0\0\0\001\0\0\0\016' > /tmp/f; mtd write /tmp/f flag; reboot

Если OpenWrt не загружается, загрузчик сам вернёт сток после исчерпания счётчика попыток.

## Nokia G-1426-MA (EcoNet EN7528, little-endian)

Другая аппаратная ревизия с похожим названием: EN7528DU, 4 ядра MIPS 1004Kc, little-endian,
Wi-Fi 6 MT7915 (2,4 + 5 ГГц), 4 × 1 GbE, USB 3.0, SPI NAND 256 МБ. Сток здесь работает из
банка B, OpenWrt ставится в банк A. **Образы G-1425 на ней не работают, и наоборот.**
Перед прошивкой проверьте `BoardInnerID=G1426MA` в `/usr/hwconfig/` стока.

- Скачать: [g1426ma-v5](https://github.com/zortald/g1425-openwrt-packages/releases/tag/g1426ma-v5) — `g1426-openwrt-v5-web-upgrade.bin` для
  веб-обновления стока (`http://192.168.1.1/upgrade.cgi`, вход `telecomadmin`) и
  `…-sysupgrade.bin` для обновления из OpenWrt (`sysupgrade -n`).
- Проверено на стоке `3FE49218HJIJ62` (HJI.J62p05).
- По умолчанию: LAN `10.100.0.1`, WAN — порт 1, открытый Wi-Fi `G1426` в обоих диапазонах.
- Настройки хранятся в томе 1 бывшего стокового `/data` (~112 МБ). Калибровка Wi-Fi
  копируется из стокового раздела config при первой загрузке.
- Вернуться в сток: `en75_chboot factory; reboot`.
- Не работает: второй PCIe-порт (MT7915 работает по одной линии), GPON, телефонный порт.
- Фид пакетов: папка [6f45af18e396b5d378e1a1baaebc5a16](6f45af18e396b5d378e1a1baaebc5a16/) (`mipsel_24kc`).
