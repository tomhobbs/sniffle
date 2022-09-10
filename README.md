# SNIFFLE

Experimental project messing about with [this](https://www.amazon.co.uk/gp/product/B08F9F276S/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

Eventually it is intended to analyse Zigbee devices but we'll play with 
Bluetooth ones first (because I have more of those to experiment with).

## Instructions

(Only tested on a single Linux laptop)

1. Install cc-tools: `sudo apt install cc-tool`

STOP: Forgot to buy a downloader cable.  

## Attemps
### Messing about with Wireshark

`dmesg` can definitely see the dongle, so it looks alright.

```
[ 6117.736994] usb 3-2: new full-speed USB device number 11 using xhci_hcd
[ 6117.888996] usb 3-2: New USB device found, idVendor=0451, idProduct=16ae, bcdDevice=86.21
[ 6117.889005] usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[ 6117.889009] usb 3-2: Product: CC2531 USB Dongle
[ 6117.889011] usb 3-2: Manufacturer: Texas Instruments
[ 6234.130396] usb 3-2: usbfs: process 26971 (cc2531) did not claim interface 0 before use
[ 6339.106148] usb 3-2: USB disconnect, device number 11
[ 6365.012091] usb 3-2: new full-speed USB device number 12 using xhci_hcd
[ 6365.164631] usb 3-2: New USB device found, idVendor=0451, idProduct=16ae, bcdDevice=86.21
[ 6365.164641] usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[ 6365.164644] usb 3-2: Product: CC2531 USB Dongle
[ 6365.164647] usb 3-2: Manufacturer: Texas Instruments
```


Needed to alter permission on WS to allow it to capture BLE traffic without
being root;

```
sudo chmod +x /usr/bin/dumpcap
```

Haven't managed to get the dongle to appear as an interface though.  

### Introducing `whsniff`

The instructions here [Github: whsniff](https://github.com/homewsn/whsniff) are
a nit out of date it seems, so I've had to modify them somewhat.

```
git clone git@github.com:homewsn/whsniff.git
cd whsniff/
git co v1.3
make
```

What's the BLE channel number?  

Run with:

```
sudo ./whsniff -c CHANNEL_NUMBER | wireshark -k -i -
```

STOP: Didn't get too far here.

## Lets try Python...

New attempt to play with the [CC2531](https://github.com/mitshell/CC2531) 
library.

Installing it was a bit of a pain - maybe because of Python versioning woes;

Anyway, swapped to Python2 to start again.

```
git clone git@github.com:mitshell/CC2531.git
cd CC2531/
virtualenv -p /usr/bin/python2 py2venv
source py2venv/bin/activate
# unalias python # Maybe this is just me?

cd ..
git clone git@github.com:mitshell/libmich.git
python2 setup.py install

cd -
source py2venv/bin/activate
pip install libusb1==1.8
pip install serial

python2 ./CC2531/sniffer.py --help
usage: sniffer.py [-h] [-d DEBUG] [-c [CHANS [CHANS ...]]] [-p PERIOD] [-n]
                  [--gps GPS] [--ip IP] [--filesock] [-f] [-s]

Use TI CC2531 dongles to sniff on IEEE 802.15.4 channels.
Forward all sniffed frames over the network (UDP/2154 port).
Each frame is packed with a list of TLV fields:
        Tag : uint8, Length : uint8, Value : char*[L]
        T=0x01, 802.15.4 channel, uint8
        T=0x02, epoch time at frame reception, ascii encoded
        T=0x03, position at frame reception (if positionning server available)
        T=0x10, 802.15.4 frame within TI PSD structure
        T=0x20, 802.15.4 frame
Output 802.15.4 frame information (channel, RSSI, MAC header, ...)

optional arguments:
  -h, --help            show this help message and exit
  -d DEBUG, --debug DEBUG
                        debug level (0: silent, 3: very verbose)
  -c [CHANS [CHANS ...]], --chans [CHANS [CHANS ...]]
                        list of IEEE 802.15.4 channels to sniff on (between 11
                        and 26)
  -p PERIOD, --period PERIOD
                        time (in seconds) to sniff on a single channel before
                        hopping
  -n, --nofcschk        displays all sniffed frames, even those with failed
                        FCS check
  --gps GPS             serial port to get NMEA information from GPS
  --ip IP               network destination for forwarding 802.15.4 frames
  --filesock            forward 802.15.4 frames to a UNIX file socket
                        /tmp/cc2531_server instead of the UDP socket
  -f, --file            output (append) frame information to file
                        /tmp/cc2531_sniffer
  -s, --silent          do not print frame information on stdout
```

Success!

It's worth noting that when run as root, Python cannot load libusb1, i.e.

```
$ sudo python2 ./CC2531/sniffer.py -d 2
ERROR: cannot import python libusb1 wrapper.
```

But now it's not finding my dongle.  So what's going on.  `CC2531.py` as some
commented out logging, so let's put that back in and see what's going on...

```
def get_CC2531():
    cc2531 = []
    ctx = usb1.USBContext()
    #
    for dev in ctx.getDeviceList(skip_on_error=True):
        if dev.getVendorID() == VID and dev.getProductID() == PID:
            LOG(' found CC2531 @ USB bus %i and address %i' \
                % (dev.getBusNumber(), dev.getDeviceAddress()))
            cc2531.append(dev)
    #
    if cc2531 == []:
        LOG(' no CC2531 found (VID %x PID %x)' % (VID, PID))
        return []
    #
    try:
        manuf = cc2531[0].getManufacturer()
    except libusb1.USBError:
        LOG(' cannot open USB device through libusb:' \
            ' add yourself in the "root" group or make an udev rule')
        return []
    # 
    return cc2531
```

Now we get a helpful message out;

```
[CC2531] cannot open USB device through libusb: add yourself in the "root" group or make an udev rule
```

So now we sort that out;

```
sudo vi /usr/lib/udev/rules.d/99-cc2531-sniffer.rules
```

And we add the rule;

```
# 'libusb' device nodes
SUBSYSTEM=="usb", \
ENV{DEVTYPE}=="usb_device", \
ATTRS{idVendor}=="0451" \
ATTRS{idProduct}=="16ae" \
MODE="0666"
```
Important: The `"16ae"` *is case sensitiveeeeeeeeee*.

The Vendor and Product IDs were both taken from the top of `CC2531.py`.


```
sudo udevadm control --reload-rules
```

And now it starts enumerating the channels and doing some sniffing!  (Maybe?)

```
$ python2 ./CC2531/sniffer.py -d 2 -f
[sniffer]  command line arguments:
Namespace(chans=[11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26], debug=2, file=False, filesock=False, gps='/dev/ttyUSB0', ip='localhost', nofcschk=False, period=1.0, silent=False)
[interpreter] server listening on ['localhost', 2154]
[GPS_reader][ERR] cannot open /dev/ttyUSB0
[CC2531] found CC2531 @ USB bus 3 and address 15
[CC2531][34337] driving CC2531 USB Dongle @ USB bus 3 & address 15, with serial 34337
[receiver][3:15:34337] forwarding to UDP socket ['localhost', 2154]
[receiver][3:15:34337] start listening on channel(s): [11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26]
[receiver][3:15:34337] sniffing on channel 11
[receiver][3:15:34337] sniffing on channel 12
...snip
```

Packets should now be visible in `/tmp/cc2531_sniffer`, which we can watch with;

```
tail -F /tmp/cc2531_sniffer
```

But I'm not getting anything, so maybe it's setup for Zigbee rather than BLE.

## References

- [Build Zigbee CC2531 Sniffer](https://community.oh-lalabs.com/t/guide-build-a-zigbee-cc2531-sniffer-how-to-use-it/469)
- [cc-tool](https://manpages.ubuntu.com/manpages/focal/en/man1/cc-tool.1.html)
- [CC2231](https://github.com/mitshell/CC2531)
- [Using Zigbee2MQTT - A Beginners Guide]( https://stevessmarthomeguide.com/using-zigbee2mqtt-beginners-guide/)
- [Github: Wireshark-cc2531](https://github.com/andrebdo/wireshark-cc2531)
- [Github: whsniff](https://github.com/homewsn/whsniff)
- [Understanding Zigbee and Wireless Mesh Networking](https://www.blackhillsinfosec.com/understanding-zigbee-and-wireless-mesh-networking/)
- [Zigbee Protocol Analyzer](https://www.offensive-wireless.com/zigbee-protocol-analyzer-sniffer/)