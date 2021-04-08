# Zigbee2mqtt

Zigbee2mqtt has support for CC2652R chip used on this board. Download the Z-Stack coordinator firmware from [@Koenkk's firmware repository](https://github.com/Koenkk/Z-Stack-firmware). The firmware you'll need can be found under `coordinator/Z-Stack_3.x.0/bin/CC2652R_coordinator_<date>.zip`, as of writing the latest version available is `CC2652R_coordinator_20210120.zip`. Download and extract this and follow the ["Flashing using BSL"](#flashing-using-bsl) instructions to burn this on your zzh.

## Configuration

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

## Troubleshooting

[This page](/radio-docs/troubleshooting/#zigbee2mqtt) contains solutions to common problems.
