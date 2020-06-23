# Configurtation of our CNC mill contolled by the Mesa 7i96 and 7i85S

## Create basic config with the 7i96 tool

Check out the installation section if not already done [7i96](installation.md#7i96-configuration-tool)

Creating a basic config using the 7i96 tool has turned out to be a good starting point for us.
But be careful, a huge downside of the tool is that it might overwrite your settings if you open your config in the tool.

## Huanyang VFD over USB/RS485

In order to communicate with the spindle over serial, you need to add the user to the dialout group

`sudo adduser <username> dialout`
  
After that, you need to logout and login again to make it work.

## HAL settings

Add these lines to your .hal file (or into a seperate one and import it)

```
# ==============================================================================
# Spindle
# ==============================================================================

# Load the Huanyang VFD user component
loadusr -Wn spindle-vfd hy_vfd -n spindle-vfd -t 1 -d /dev/spindle0 -p none -r 38400 -s 1

# Spindle signales via serial connection
setp spindle-vfd.enable 1
net spindle-fwd spindle.0.forward => spindle-vfd.spindle-forward
net spindle-reverse spindle.0.reverse => spindle-vfd.spindle-reverse
net spindle-speed-cmd  spindle.0.speed-out-abs => spindle-vfd.speed-command
net spindle-on spindle.0.on => spindle-vfd.spindle-on
net spindle-at-speed spindle.0.at-speed => spindle-vfd.spindle-at-speed

setp pid.3.Pgain       [SPINDLE]P
setp pid.3.Igain       [SPINDLE]I
setp pid.3.Dgain       [SPINDLE]D
setp pid.3.bias        [SPINDLE]BIAS
setp pid.3.FF0         [SPINDLE]FF0
setp pid.3.FF1         [SPINDLE]FF1
setp pid.3.FF2         [SPINDLE]FF2
setp pid.3.deadband    [SPINDLE]DEADBAND
setp pid.3.maxoutput   [SPINDLE]MAX_OUTPUT
setp pid.3.maxerror    [SPINDLE]MAX_ERROR
```

Make sure `loadrt pid num_chan=4` is set to 4 or more in order to have a PID for the spindle.

### INI settings

Add these lines to your .ini
```
# ==============================================================================
# Parameters for the Spindle
# ==============================================================================

[SPINDLE]
SPINDLE_TYPE = openLoop
DEADBAND     = 0
P            = 50
I            = 200
D            = .2
FF0          = 0
FF1          = 0
FF2          = 0
BIAS         = 0
MAX_OUTPUT   = 0
MAX_ERROR    = 50
SCALE        = 6000
MINLIM       = 0
MAXLIM       = 6000
```

### udev rules

In order to get a device for the spindle (`/de/spindle0`) you can create a custom udev rule.
To do so create a file named `99_mill.rules` in `/lib/udev/rules.d/`.
In this file you put the following rule:

```
SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="00000000", SYMLINK+="spindle0"
```

You might have to adjust 0403 and 6001 to fit your USB/RS485 converter. To find the correct values, type `lsusb`.
The output can look like that:

```
Bus 004 Device 002: ID 8087:8000 Intel Corp. 
Bus 004 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 8087:8008 Intel Corp. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 007: ID 1a86:7523 QinHeng Electronics HL-340 USB-Serial adapter
Bus 002 Device 006: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 002 Device 009: ID 7392:7811 Edimax Technology Co., Ltd EW-7811Un 802.11n Wireless Adapter [Realtek RTL8188CUS]
Bus 002 Device 010: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

The `Bus 002 Device 010: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC` is our converter, after THE ID you see the two numbers you need.

You need to reload udev with `sudo udevadm trigger` in order to load the rule.

A `ls -la /dev/spindle0` should give you:

```
lrwxrwxrwx 1 root root 7 Jun 23 16:05 /dev/spindle0 -> ttyUSB1
```
