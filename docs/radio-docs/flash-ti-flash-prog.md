title: Flash Firmware using Flash Prog 2

# Flash Firmware using TI SmartRF Flash Programmer v2

## Download SmartRF Flash Programmer v2

Head over to [TI's website](https://www.ti.com/tool/FLASH-PROGRAMMER) and download a copy of SmartRF Flash Programmer v2 (also refered to as: FLASH-PROGRAMMER-2). You'll need to register for an account and request approval for the download but it's a quick and easy process.

Make sure you download FLASH-PROGRAMMER-2, *not FLASH-PROGRAMMER*, as the latter does not support the CC2652 devices used on the Electrolama sticks.

Install SmartRF Flash Programmer v2 and fire it up.

## Put your stick in BSL mode

Please follow the following instructions to put your stick in BSL mode:

  - Unplug your stick from the host
  - Press the `BSL` pushbutton and keep holding while plugging the device back into the host:

![zzh BSL pushbutton](/_assets/zzh-bsl-button.jpg)

  - Give it a few seconds for the device to settle and set up and release the BSL button
  - Your stick should now be in ROM bootloader mode

<ins>It is imperative that you press and hold the BSL pushbutton **before** plugging it in to the host and release it **after a few seconds**. If you don't follow this, your stick will not enter BSL mode and flashing will fail.</ins>


## Identify and select serial port

We need to identify what serial port to use, right click on the Start menu and go to the "Device Manager". Plug your stick in and look under "Ports (COM & LPT)" for the COM Port it will pick up:

![Serial Ports in Windows Device Manager](/_assets/zzh-port-windows.png)

If you don't see your stick under "Ports (COM & LPT)" with a COM Port assigned, you may need to install the drivers for it (see [here](/radio-docs/drivers/)).

With the COM Port identified in "Device Manager", switch over to SmartRF Flash Programmer v2 and click on "Unknown" displayed under the serial port you've identified:

![COM Port selection in Flash Programmer 2](/_assets/flash-prog-port.png)


## Select correct device

Next up: We need to tell SmartRF Flash Programmer v2 what target device we have on our stick. For zzh, select "CC2652R" and for zzhp/zzhp-lite, select "CC2652P":

![Flash Programmer 2 Chip Selection](/_assets/flash-prog-device.png)


## Select firmware file and flash device

With the correct serial port chosen and target device selected, click "Browse" and select the firmware you want to flash (see [here](/radio-docs/#step-2-download-the-correct-firmware-for-your-stick) for help on choosing firmware).

<p class="warn">⚠️ <b>WARNING:</b> It is crucial that you download the correct firmware for your stick as using the wrong firmware will disable the BSL and you will need an external debugger / programmer to flash your stick again. Instructions for that can be found <a href="/radio-docs/advanced/flash-jtag/">here</a>.</p>

With the correct firmware chosen, make sure that "Erase", "Program" and "Verify" are all chosen under "Actions" and press the blue button to start flashing:

![Flash Programmer 2 GUI](/_assets/flash-prog.png)

If all goes to plan, after a few seconds you should see a green bar confirming that the flashing operating suceeded:

![Successful flashing using Flash Programmer 2](/_assets/flash-prog-ok.png)

You can now proceed to configuring your software of choice (see [here](/radio-docs/#step-4-setup-zigbee2mqtt-or-zha)).


## Troubleshooting

### Error: "No response from device"

If your stick is not in BSL mode or you have not chosen the correct serial port, you will get the following error message:

![BSL Mode failure](/_assets/flash-prog-fail.png)

Unplug your stick and follow instructions above to try again.

### Error: "Failed to open file"

If your path or filename contains extended characters, Flash Programmer 2 will fail. Simply renaming the file/folder or just moving your firmware file to a simple path (e.g: "C:\firmware.hex") will fix this issue:

![Extended characters in path/filename failure](/_assets/flash-prog-ext-chars.png)



