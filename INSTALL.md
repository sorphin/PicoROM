# PicoROM Installation Guide

This guide provides detailed instructions on how to set up and install the PicoROM software and firmware.

## General Installation

To use a PicoROM, you will need the `picorom` command-line application. This is the software the communicates with the PicoROM, manages the firmware and uploads ROM images.

It is available as part of the [latest release](https://github.com/wickerwaka/PicoROM/releases).

1.  **Download the Release:** Download the archive corresponding to your operating system.
    * **Windows:** [Intel 64-bit](https://github.com/wickerwaka/PicoROM/releases/latest/download/picorom-x86_64-pc-windows-msvc.exe)
    * **MacOS:** [ARM 64-bit](https://github.com/wickerwaka/PicoROM/releases/latest/download/picorom-aarch64-apple-darwin), [Intel 64-bit](https://github.com/wickerwaka/PicoROM/releases/latest/download/picorom-x86_64-apple-darwin)
    * **Linux:** [Intel 64-bit](https://github.com/wickerwaka/PicoROM/releases/latest/download/picorom-x86_64-unknown-linux-musl), [ARM 32-bit](https://github.com/wickerwaka/PicoROM/releases/latest/download/picorom-armv7-unknown-linux-musleabihf)
2.  **Rename:** You may want to rename the file to just `picorom` (or `picorom.exe` on Windows) for easier access in the terminal.
3.  **Install:** Copy the `picorom` executable to a location in your system path (e.g., `/usr/local/bin` on Linux/MacOS or a dedicated folder included in your `PATH` on Windows).

## Firmware
The `picorom` tool includes embedded firmware for all hardware variants. To update a device running firmware 2.0 or later you use the `picorom firmware` command with the device name (`example_device` in this case):

```console
foo:~ $ picorom firmware example_device
```

This will display available firmware versions and flash the selected one. You can also specify a firmware file directly:

If you have a device with the 1.0 firmware or simply one that has no firmware installed you can use the `picorom firmware` command without any device name. This will look for any connected RP2040-based device and flash the firmware to it.

After flashing, power-cycle the device and verify with `picorom list` and `picorom get <name> version`.


---

## Platform-Specific Concerns

### MacOS: Security and Gatekeeper
Because the `picorom` tool is not signed by a registered Apple developer, MacOS Gatekeeper will prevent it from running initially.

* **To authorize the app:**
    1.  Open the terminal and attempt to run `picorom`. You will see a security warning.
    2.  Go to **System Settings > Privacy & Security**.
    3.  Scroll down to the "Security" section and look for the message saying `picorom was blocked from use because it is not from an identified developer`.
    4.  Click **Open Anyway**.
    5.  Alternatively, you can right-click (or Control-click) the application in Finder and select **Open**, then click **Open** again in the confirmation dialog.

### Windows: Zadig and WinUSB Drivers
If you wish to use the firmware update feature directly through the `picorom` tool (which uses the PICOBOOT interface), Windows may require a specific driver. By default, Windows may not associate the RP2040's bootloader interface with the correct driver.

* **To install the driver:**
    1.  Download and run [Zadig](https://zadig.akeo.ie/).
    2.  Put your PicoROM into **BOOTSEL** mode (try to update with the `picorom firmware` command, it will fail and leave the device in BOOTSEL mode).
    3.  In Zadig, go to **Options > List All Devices**.
    4.  Select **RP2 Boot (Interface 1)** from the dropdown menu.
    5.  Ensure **WinUSB** is selected as the replacement driver.
    6.  Click **Install Driver** (or **Replace Driver**).

*Reference: This process is standard for RP2040 devices and is documented in the Raspberry Pi Foundation's [Getting Started with Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf) guide (Appendix B).*

### Linux: udev Rules for Non-Root Access
By default, Linux requires root privileges (`sudo`) to access USB devices. To use `picorom` as a regular user, you must create a udev rule to allow access to devices with the Raspberry Pi Vendor ID (**0x2e8a**).

* **To set up the rule:**
    1.  Create a new rules file:
        ```bash
        sudo nano /etc/udev/rules.d/99-pico.rules
        ```
    2.  Add the following line to the file:
        ```text
        SUBSYSTEM=="usb", ATTRS{idVendor}=="2e8a", TAG+="uaccess"
        ```
    3.  Save the file and reload the udev rules:
        ```bash
        sudo udevadm control --reload-rules
        sudo udevadm trigger
        ```
    4.  Unplug and reconnect your PicoROM for the changes to take effect.

*Reference: For more complex setups or alternative rule configurations, refer to the [Raspberry Pi USB PID repository](https://github.com/raspberrypi/usb-pid).*

---

## BOOTSEL mode
If a PicoROM becomes unresponsive as part of the firmware update process you can force it into it's bootloader (BOOTSEL) mode by bridging the `USB` and `GND` pads on the header near the RP2040 while powering on.

![USB Boot header location](docs/usb_boot.jpg)

