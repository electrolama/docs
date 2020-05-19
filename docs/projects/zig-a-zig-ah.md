title: zig-a-zig-ah! (CC2652 Stick)

# Overview

![zzh PCBA](/_assets/zzh-pcba-green.jpg)


 zig-a-zig-ah! (zzh, for short) is a tiny "USB stick" form-factor development board for multiprotocol RF tinkering.

 It features:

 - TI [CC2652R](http://www.ti.com/product/CC2652R) 2.4 GHz multi-protocol wireless microcontroller targeting Thread, Zigbee, Bluetooth 5 Low Energy, IEEE 802.15.4g, IPv6-enabled smart objects (6LoWPAN) and proprietary systems
 - Communicates with the host computer via the common CH340 USB-UART bridge, no manual driver installation needed in most cases (Windows and Linux)
 - Self-programming via the TI [CC-series serial bootloader](http://www.ti.com/lit/an/swra466c/swra466c.pdf). As long as it is not explicitly disabled in code, no external programmer needed! Pushbutton on the default pin to trigger this mode
 - cJTAG debug header, in case you disable BSL by accident or want a proper debug interface
 - SMA antenna port for an external antenna of your choice
 - General purpose LED

It is designed to fit in a tiny *[Gōng mó](https://www.theatlantic.com/technology/archive/2014/05/chinas-mass-production-system/370898/)* "USB Shell" and looks a bit like this when paired with your favourite SBC:

![zzh plugged in to Raspberry Pi](/_assets/zzh-plugged-in.jpg)

Think of it as an upgrade to the ubiquitous [CC2531 USB Sticks](https://www.google.com/search?q=cc2531+stick) commonly used for Zigbee tinkering. CC2652 has a much beefier processor, more memory and a sane free compiler that should enable easier development compared to the old 8051 based CC2530/1 devices.


## Flavours

There are two versions of zzh:

  - **zzh** (CC2652R) Revision A is released and a batch of boards have been produced. See below for puchasing details.
  - **zzh-p** is the experimental version with CC2652P (built-in PA). Shares most of the same design with zzh except for the RF parts and will fit in a slightly larger case. It is a work in progress, sign up to the [Electrolama mailing list](https://mailchi.mp/1746be86dd81/electrolama) to be notified of project updates and when kits/assembled units go on sale. A sneak peek:

![zzh-p preview](/_assets/zzh-p-preview.png)

[Click here](#aside-ti-part-numbers) for a summary of different chips mentioned, if you are confused about all these part numbers.

## Purchase 

Assembled versions of zzh will be available on the [Electrolama Tindie Store](https://www.tindie.com/stores/electrolama/) very soon. Orders ship from London, UK.

Each zzh order contains a fully assembled and tested PCBA along with a plastic enclosure and a small antenna:

![zzh case and antenna](/_assets/zzh-case-antenna.jpg)

An optional debug adapter kit of parts (requires assembly) can also be purchased:

![zzh debug adapter kit](/_assets/zzh-debug-adapter-kit.jpg)

A portion of each sale will be donated to [@Koenkk](https://github.com/Koenkk/) to support his work on [Zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt) and the [public firmware images](https://github.com/Koenkk/Z-Stack-firmware) used by zzh and many other projects.


## User Manual

### Important Note

**Please keep in mind that zzh is a general purpose development board and as such, it is shipped "blank" with no code on the wireless microcontroller. You will need to program it before it does anything meaningful.** Given the application dependent nature of this board, limited after-sales support can be provided for Tindie purchases (For example: we can help you program this board but can't fix your Zigbee network range issues or offer specific software support). 

### Drivers for CH340

The USB-Serial functionality is handled all externally by CH340 and as far as CC2652 is concerned, communication happens over straight UART. First step in getting set up with zzh is to ensure that the host computer has the right drivers for the CH340 installed.

Plug your device in (don't worry about the firmware just yet) and ensure that zzh is recognised by your operating system before proceeding to the flashing steps.

#### Linux

Issue `dmesg` and observe the device enumeration:

```Shell
[152343.203201] usb 1-1.4: new full-speed USB device number 5 using dwc_otg
[152343.336384] usb 1-1.4: New USB device found, idVendor=1a86, idProduct=7523, bcdDevice= 2.62
[152343.336400] usb 1-1.4: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[152343.336409] usb 1-1.4: Product: USB2.0-Serial
[152343.338315] ch341 1-1.4:1.0: ch341-uart converter detected
[152343.341440] usb 1-1.4: ch341-uart converter now attached to ttyUSB0
```

CH340 drivers have been in the kernel for a while and as long as you are using a relatively new version ([driver changelog](https://github.com/torvalds/linux/commits/master/drivers/usb/serial/ch341.c)), you shouldn't have any issues.

#### Windows

The drivers for CH340 should be automatically be picked up and you should see your device under "Ports (COM & LPT)" in Device Manager:

![serial port windows](/_assets/zzh-port-windows.png)

If you need to install the drivers manually, head over [here](http://www.wch.cn/downloads/CH341SER_ZIP.html) for the official drivers.

#### macOS

(tbd)

### Flashing using BSL

One of the nice things about these new chips from TI is the inclusion of a serial bootloader in the ROM. Enabled by default and available unless explicitly modified by the application (see `CCFG:BL_CONFIG`), you can flash code without needing an external programmer. In fact, if the device is empty (either fresh from the factory or after an erase cycle), it automatically enters the bootloader mode.

A summary of when the bootloader is invoked can be seen in this flowchart below:

![bsl flowchart](/_assets/bsl-flowchart.png)

(["CC2538/CC26x0/CC26x2 Serial Bootloader Interface"](http://www.ti.com/lit/an/swra466c/swra466c.pdf), Figure 1)

The default "backdoor enable" pin on CC2652R is `DIO_13` ([not `DIO_11` as noted in SWRA466C!](https://e2e.ti.com/support/wireless-connectivity/bluetooth/f/538/t/871984?LAUNCHXL-CC26X2R1-Bootloader-backdoor-enable-pin)) and zzh has a small pushbutton on this pin to make tinkering with it easier.

To trigger the ROM bootloader, follow these steps:

  - Unplug zzh from the host
  - Press the `BSL` button on the board and keep holding while plugging the device back into the host:

![zzh BSL pushbutton](/_assets/zzh-bsl-button.jpg)

  - Give it a few seconds for the device to settle and set up and release the BSL button
  - zzh should now be in ROM bootloader mode

[TI CC13xx/CC2538/CC26xx Serial Boot Loader (cc2538-bsl for short)](https://github.com/JelmerT/cc2538-bsl) is a Python script that can be used to flash the board and should work fine on pretty much any OS.

#### Linux
To run cc2538-bsl.py you need to have python and pip installed on your system. If you don't have them installed, refer to your distribution package manager to get it set up. (On Debian/Ubuntu, `sudo apt update && sudo apt-get install python3-pip` should work. `sudo pip ...` is not the optimal way of installing packages but Python package management is out of scope for this document.)

##### Download and extract cc2538-bsl

``` bash
wget -O cc2538-bsl.zip https://codeload.github.com/JelmerT/cc2538-bsl/zip/master && unzip cc2538-bsl.zip
```
##### Install required dependencies 

```bash
sudo pip3 install pyserial intelhex
```

##### Flash firmware

At this point you are going to need a firmware file to flash to your device. As a test, you can grab [this small program (blink.bin)](/_assets/blink.bin) that toggles the LED connected to `DIO_7`.

To flash `blink.bin`, run:

```Shell
$ ./cc2538-bsl.py -p /dev/ttyUSB0 -evw blink.bin
```

**Don't forget to change `/dev/ttyUSB0` to the port used on your machine!**

This shouldn't take long and if successful, you will see blinkenlights!

If you want to re-flash zzh with new firmware, put it in bootloader mode and follow the flashing instructions. **This will work as long as BSL is not disabled by the firmware you burn to these devices.**

##### Erase flash

To completely erase the device flash, run:

```Shell
$ ./cc2538-bsl.py -p /dev/ttyUSB0 -e
```

#### Windows
To run cc2538-bsl.py you need to have Python installed on your system. Download [Python for Windows](https://www.python.org/downloads/) and install. After installation verify Python is correctly installed by running `python -V` in Command Prompt. It should return Python and the version number.

If you receive a message similiar to: `Python is not recognized as an internal or external command, operable program or batch file.` means Python is either not installed or the system variable path hasn’t been set. You’ll need to launch Python from the folder in which it is installed or adjust your system variables to allow it to be launched from any location.

###### Install required dependencies 
Open Command Prompt and check if `pip` is installed by running `pip -V` which will show its version and install location.

From the same Command Prompt run:

```bash
pip3 install pyserial intelhex
```

###### Download and extract cc2538-bsl

Download the [zipped code](https://github.com/JelmerT/cc2538-bsl/archive/master.zip) and extract to a folder.


###### Flash firmware

At this point you are going to need a firmware file to flash to your device. As a test, you can grab [this small program (blink.bin)](/_assets/blink.bin) that toggles the LED connected to `DIO_7`. It is recommended to download it to the same folder cc2538-bsl is extracted to.

To flash `blink.bin`, navigate to the folder where you extracted cc2538-bsl and open a Command Prompt there

```bash
cc2538-bsl.py -p COM14 -evw blink.bin
```

**Don't forget to change `COM14` to the port used on your machine!**

This shouldn't take long and if successful, you will see blinkenlights!

If you want to re-flash zzh with new firmware, put it in bootloader mode and follow the flashing instructions. **This will work as long as BSL is not disabled by the firmware you burn to these devices.**

##### Erase flash

To completely erase the device flash, run:

```bash
cc2538-bsl.py -p COM14 -e
```

_Example of a successful flash of Z-Stack coordinator firmware under Windows:_
![zzh flash complete](/_assets/zzh-flash-complete.jpg)

#### macOS
To run cc2538-bsl.py you need to install some extra python dependancies, python3 should already be shipped with macOS (Catalina).

##### Download and extract cc2538-bsl

``` bash
wget -O cc2538-bsl.zip https://codeload.github.com/JelmerT/cc2538-bsl/zip/master && unzip cc2538-bsl.zip
```
##### Install required dependencies

As we cannot write to the system's location we need to install the dependancies with in the user location.
The downside is this has to be installed for every user that will use cc2538-dsl.

```bash
$ /usr/bin/python3 -m pip install --user pyserial intelhex
```

##### Flash firmware

At this point you are going to need a firmware file to flash to your device. As a test, you can grab [this small program (blink.bin)](/_assets/blink.bin) that toggles the LED connected to `DIO_7`.

To flash `blink.bin`, run:

```bash
$ ls -l /dev/tty.usbserial-*
$ ./cc2538-bsl.py -p /dev/tty.usbserial-2212420 -evw blink.bin
```

**Don't forget to change `/dev/tty.usbserial-2212420` to the port used on your machine!**

This shouldn't take long and if successful, you will see blinkenlights!

If you want to re-flash zzh with new firmware, put it in bootloader mode and follow the flashing instructions. **This will work as long as BSL is not disabled by the firmware you burn to these devices.**

##### Erase flash

To completely erase the device flash, run:

```bash
$ ./cc2538-bsl.py -p /dev/tty.usbserial-2212420 -e
```

### Flashing using external debugger

**Please Note**: For general purpose usage, you will not need an external debugger and BSL method discussed above will be enough for your flashing needs. If you are a firmware developer needing proper debugging facilities and/or want to get access to the RF tools in TI SmartRF Studio, follow along.

Due to space constraints, the debug header on zzh is (sadly) non-standard. 

#### Pinout for the zzh debug header

![zzh debug header pinout](/_assets/zzh-debug-pinout.png)

#### zzh Debug Adapter

To convert the zzh 5-pin debug header to a standard 10-pin Cortex Debug Connector, a small adapter PCB is needed (unless you hack and solder an IDC cable to the board directly). Design files for this adapter can be found [in the repo](https://github.com/electrolama/zig-a-zig-ah/tree/master/Debug-Adapter).

With this adapter board plugged in to zzh, you can connect any debugger you wish to get access to the cJTAG port. This little adapter board kit (some soldering required) is available as an optional add-on to zzh on [Electrolama Tindie Store](https://www.tindie.com/stores/electrolama/).

The cheapest option for an officially supported debugger is the [CC-DEVPACK-DEBUG](http://www.ti.com/tool/CC-DEVPACK-DEBUG), available from most common electronics distributors.

![The Debug Sandwich](/_assets/zzh-debugger-devpack.jpg)

(The Debug Sandwich: CC-DEVPACK-DEBUG on top of the zzh-debug-adapter on top of zzh.)

Here'a handy pinout reference for CC-DEVPACK-DEBUG:

![CC-DEVPACK-DEBUG Pinout](/_assets/cc-devpack-debug-pinout.png)

### Zigbee2mqtt

Zigbee2mqtt has support for CC2652R chip used on this board, for now still in development phase. Download the Z-Stack coordinator firmware from [@Koenkk's firmware repository](https://github.com/Koenkk/Z-Stack-firmware/tree/develop).

As of writing, the latest development firmware available is: [CC26X2R1_20200417.zip](https://raw.githubusercontent.com/Koenkk/Z-Stack-firmware/develop/coordinator/Z-Stack_3.x.0/bin/CC26X2R1_20200417.zip). Download and extract this and follow the ["Flashing using BSL"](#flashing-using-bsl) instructions to burn this on your zzh.

#### Configuration

Make sure that the CH340 USB-serial bridge drivers are installed and your device is recognised ([instructions here](#drivers-for-ch340)).

With the correct serial port in use identified, edit your Zigbee2mqtt `configuration.yaml`:

```yaml
serial:
  port: /dev/ttyUSB0  (change this if it is different on your machine)
advanced:
  rtscts: false
```

The `rtscts: false` directive does not appear in the default configuration file but is crucial for the operation of zzh so please **don't ignore that**. 
Also note that it sits in the `advanced:` config group and not in `serial:`.

#### Troubleshooting

**"Error: Failed to connect to the adapter (Error: SRSP - SYS - ping after 6000ms)"**

This is a general error if there are communication problems with the adapter used.

A checklist to go through if you get this error message:

  * Try replugging zzh once or twice.
  * Make sure you have the correct serial port and `rtscts: false` option set in your `configuration.yaml`
  * Make sure you have the correct firmware burned on your adapter. **Remember, zzh boards ship blank** so you will need to burn the appropriate firmware before using it with any application.
  * Are you using any virtualisation/container pass-through for the USB device? There have been reports of these potentially causing problems so for debugging purposes try communicating with the adapter directly from the host OS it is connected to (i.e: see if it works without the bypass)
  * Make sure you are using an up-to-date version of Zigbee2mqtt on the host and firmware on the adapter.

### Zigbee Home Automation for Home Assistant

[Zigbee Home Automation](https://www.home-assistant.io/integrations/zha/) or ZHA is an integration for Home Assistant. It uses the same Z-Stack coordinator firmware as Zigbee2mqtt and connects zzh directly with Home Assistant.    
**Support for TI chips (especially CC2562) is still in development stage!**

#### Configuration

Make sure zzh is recognised and available to your Home Assistant instance. Navigate to _Configuration -> Integrations_, click on the `+` icon, find _Zigbee Home Automation_ and click on it.

![ZHA Integration](/_assets/zzh-zha-install.png)

Choose the device path of zzh and wait for installation to complete. 
navigate to the ZHA integration at the bottom of the Configuration page. You will now have "Texas Instruments ZNP Coordinator" as the first device in ZHA. 

![ZHA Coordinator](/_assets/zzh-zha-coord.png)

In case the autodetection fails, a manual setup menu will be displayed. Check the device path and set _Radio Type_ as **ti_cc**. Leave other options as they are.

If ZHA is unable to connect to zzh try replugging zzh or change to another USB port.

## Aside: TI Part Numbers

TI has quite a few different chips and they are all used/mentioned in open-source Zigbee world which can be daunting if you are just starting out. Here's a quick summary of part numbers and key features, starting with the older generation:

  - [CC2530](https://www.ti.com/product/CC2530): 2.4GHz Zigbee and IEEE 802.15.4 wireless MCU. 8051 core, has very little RAM. Needs expensive compiler license for official TI stack.
  - [CC2531](https://www.ti.com/product/CC2531): CC2530 with built-in USB. Used in the cheap "Zigbee sticks" sold everywhere.

The newer generation:

  - [CC2652R](https://www.ti.com/product/CC2652R): 2.4GHz only multi-protocol (Zigbee, Bluetooth, Thread, ...) wireless MCU. Cortex-M0 core for radio stack and Cortex-M4F core for application use, plenty of RAM. Free compiler option from TI. **zzh uses this chip**.
  - [CC2652RB](https://www.ti.com/product/CC2652RB): Pin compatible "Crystal-less" CC2652R (so you could use it if you were to build your own zzh and omit the crystal) but **not firmware compatible**.
  - [CC2652P](https://www.ti.com/product/CC2652P): CC2652R with a built-in RF PA. Not pin or firmware compatible with CC2652R/CC2652RB. **zzh-p will use this**.

  Multi frequency chips:

  - [CC1352R](https://www.ti.com/product/CC1352R): Sub 1 GHz & 2.4 GHz wireless MCU. Essentially CC2652R with an extra sub-1GHz radio.
  - [CC1352P](https://www.ti.com/product/CC1352P): CC1352R with a built in RF PA.

Auxiliary chips:

  - [CC2591](https://www.ti.com/product/CC2591) and [CC2592](https://www.ti.com/product/CC2592): 2.4 GHz range extenders. **These are not wireless MCUs, just auxillary RF PA and LNA in the same package.**

## Downloads

  - EAGLE source files in [electrolama/zig-a-zig-ah](https://github.com/electrolama/zig-a-zig-ah)
  - [Schematic (pdf), Revision A](/_assets/zzh-revA-schematic.pdf)

## Changelog

In the repo, [click here](https://github.com/electrolama/zig-a-zig-ah/blob/master/CHANGELOG.md).

## License

zig-a-zig-ah! is designed by Electrolama / Omer Kilic and licensed under the [Solderpad Hardware License 2.0](https://solderpad.org/licenses/SHL-2.0/). 

### Regulatory Notice

This kit is designed to allow Product developers to evaluate electronic components, circuitry, or software associated with the kit to determine whether to incorporate such items in a finished product and Software developers to write software applications for use with the end product. This kit is not a finished product and when assembled may not be resold or otherwise marketed unless all required FCC (or any other local authority) equipment authorizations are first obtained. Operation is subject to the condition that this product not cause harmful interference to licensed radio stations and that this product accept harmful interference.

## ACKs

Thanks to:

  - [@GeorgeIoak](https://twitter.com/GeorgeIoak) and [@matthewvenn](https://twitter.com/matthewvenn) for design review and comments during early prototyping
  - [Fredrik K](https://www.linkedin.com/in/fredrik-kervel/) for his domain expertise, design review and (much appreciated!) RF help
  - [@KoenKK](https://github.com/koenkk/) for his work on the great Zigbee2mqtt project and testing prototypes
  - [@egelmex](https://twitter.com/egelmex) for testing prototypes, improving docs and being an all around cool dude

Name credit goes to [@9600](https://twitter.com/9600/), this had a much boring name before he suggested zig-a-zig-ah!

## Contact 

For general enquiries, suggestions and errors spotted: Email us at hello@this-domain. Community contributions to these pages are very much encouraged so you could also send pull requests on the [documentation repo](https://github.com/electrolama/docs) (source of these pages) with your proposed changes.

For support regarding Tindie purchases: support@this-domain.
