# ~~Precision 5490~~ ThinkPad T16s Gen 6 AMD Linux notes

- keyd: find ID of internal keyboard with `keyd monitor`. Remember to check the list - the keyd virtual keyboard will show your keystrokes.
- Need to fix a huge regression in libinput where thumb detection is much less useful...not sure why nobody seemed to notice that
- Internal microphone has a bad DC offset and low signal level. Not sure where to start with this; might boot Windows to test

## Suspend issue
This was difficult to troubleshoot. It's possible not all of these steps were necessary.

The first thing I tried was `cat /proc/acpi/wakeup` and disabling all those, but that did not make a difference.

My last steps before getting it to work correctly were these:
`echo "disabled" | sudo tee /sys/bus/i2c/devices/i2c-ELAN901C\:00/power/wakeup`
`echo "disabled" | sudo tee /sys/bus/i2c/devices/i2c-ELAN0676\:00/power/wakeup`

Those two i2c devices are the touchscreen (901C) and what is listed in dmesg as a touchpad and a mouse (0676).

It's possible only one of those is necessary. Those steps were derived from https://askubuntu.com/questions/1133919/ubuntu-18-04-2-immediately-wakes-up-from-suspend but I also checked the i2c bus since everything in USB was already disabled.

For now I created a sleep hook, `/usr/lib/systemd/system-sleep/disable-wakeup`:

```
#!/bin/sh

case "$1" in
    pre)
        echo "disabled" | tee /sys/bus/i2c/devices/i2c-ELAN901C:00/power/wakeup
        echo "disabled" | tee /sys/bus/i2c/devices/i2c-ELAN0676:00/power/wakeup
        ;;
esac
```
