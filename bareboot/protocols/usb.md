---
layout: default
title: "USB CDC & DFU Touch"
nav_order: 4
parent: "Protocols & Transports"
grand_parent: "BareBoot"
permalink: /bareboot/protocols/usb/
description: "USB bootloader transport: CDC 1200-baud touch reset, DFU Detach, and vendor control endpoints"
---

# USB CDC & DFU Touch Protocols

For microcontrollers equipped with native USB 2.0 Full-Speed or High-Speed peripherals (such as RP2350, RP2040, STM32, and CH32V203), BareBoot provides instant, driverless firmware update capabilities across Linux, macOS, and Windows.

---

## 1. USB CDC 1200-Baud Touch Protocol

The 1200-baud touch convention is the industry standard mechanism for rebooting USB virtual COM port devices into bootloader mode without hardware buttons:

1. **Host Connection**: The host software opens the USB serial port (`/dev/ttyACM0` or `COMx`).
2. **Line Coding Change**: The host issues a USB CDC `SET_LINE_CODING` class request configuring:
   - `dwDTERate = 1200`
   - `bCharFormat = 1` (1 stop bit)
   - `bParityType = 0` (None)
   - `bDataBits = 8`
3. **DTR Deassertion**: The host clears the Data Terminal Ready (`DTR`) control line via `SET_CONTROL_LINE_STATE` (`wValue = 0`).
4. **Application Interception**:
   ```rust
   //! Handling 1200-baud touch in Embassy USB CDC ACM class
   if line_coding.baud_rate() == 1200 {
       unsafe {
           enter_bareboot_cortex_m(0x0800_0000);
       }
   }
   ```
5. **Re-Enumeration**: The device resets and re-enumerates as the BareBoot CDC flasher device or Katapult USB bootloader.

---

## 2. USB DFU (Device Firmware Upgrade) Class Detach

For applications implementing the USB DFU runtime interface (`bInterfaceClass = 0xFE, bInterfaceSubClass = 0x01`):

- **Control Request**:
  - `bmRequestType`: `0x21` (Host to Device, Class, Interface)
  - `bRequest`: `DFU_DETACH` (`0x00`)
  - `wValue`: Timeout in milliseconds
  - `wIndex`: Interface Number
  - `wLength`: `0`
- **Application Response**: The application acknowledges the setup transaction, waits for USB bus suspend or writes the cross-reset mailbox token `0x5984E3FA6CA1589B`, and executes `SCB::sys_reset()`.

---

## 3. USB Vendor Control Transfers

For non-CDC and non-DFU applications (such as custom HID, WinUSB, or bulk instrumentation pipes), BareBoot defines a dedicated vendor control setup packet:

| Setup Field | Value | Meaning |
|---|---|---|
| `bmRequestType` | `0x40` | Host-to-Device, Vendor Request, Target Device |
| `bRequest` | `0xBB` | `BAREBOOT_CMD` |
| `wValue` | `0x0001` | `CMD_ENTER_BOOTLOADER` |
| `wIndex` | `0x0000` | Reserved |
| `wLength` | `0x0000` | No data phase |

When received on Endpoint 0, the device initiates the standard mailbox handoff and resets.
