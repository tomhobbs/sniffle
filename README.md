# SNIFFLE

A small project just containing instructions and scripts relating to different
hardware dongles, so far;

- [CC2531 (Zigbee sniffer)](https://www.amazon.co.uk/gp/product/B7F9F276S/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)

Eventually it is intended to analyse Zigbee devices but we'll play with 
Bluetooth ones first (because I have more of those to experiment with).

# SNIFFLE


## Firmware

Flashing the firmware required cc-tool.

```
# First install the following
sudo apt-get install autoconf
sudo apt install libtool
sudo apt install libboost-all-dev
```

Then complete the instructions here: [Flashing the CC2531](https://www.zigbee2mqtt.io/guide/adapters/flashing/flashing_the_cc2531.html#linux-or-macos)

Then using the downloaded cc-tool program;

```
# Reset the firmware back to TI's sniffer
sudo <PATH_TO_PROJ>/cc-tool -e -w <PATH_TO_FIRMWARE>/sniffer_fw_xx2531.hex
```

### CC2531

| Device | File | Description |
|---|---|---
| CC2531 | sniffer_fw_cc2531.hex | Default sniffer firmware from TI |

## References

- [Build Zigbee CC2531 Sniffer](https://community.oh-lalabs.com/t/guide-build-a-zigbee-cc2531-sniffer-how-to-use-it/469)
- [cc-tool](https://manpages.ubuntu.com/manpages/focal/en/man1/cc-tool.1.html)
- [CC2231](https://github.com/mitshell/CC2531)
- [Using Zigbee2MQTT - A Beginners Guide]( https://stevessmarthomeguide.com/using-zigbee2mqtt-beginners-guide/)
- [Github: Wireshark-cc2531](https://github.com/andrebdo/wireshark-cc2531)
- [Github: whsniff](https://github.com/homewsn/whsniff)
- [Understanding Zigbee and Wireless Mesh Networking](https://www.blackhillsinfosec.com/understanding-zigbee-and-wireless-mesh-networking/)
- [Zigbee Protocol Analyzer](https://www.offensive-wireless.com/zigbee-protocol-analyzer-sniffer/)
- [TI CC Debugger](https://www.ti.com/lit/ug/swru197h/swru197h.pdf?ts=1665100266137)
- [Flashing the CC2531](https://www.zigbee2mqtt.io/guide/adapters/flashing/flashing_the_cc2531.html#linux-or-macos)