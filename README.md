# Bluetooth_Based_Digital_Notice_Board

An embedded systems project featuring a wireless digital notice board driven by the **NXP LPC2129 (ARM7)** microcontroller. The system receives real-time text updates from an Android smartphone via Bluetooth, stores them in non-volatile EEPROM memory to prevent data loss on reboot, and displays smooth scrolling announcements across an 8×8 LED dot matrix display panel.

---

## Overview

Traditional notice boards require manual pinning or physical access to change announcements. This project replaces static boards with an automated, wireless display unit. An authorized user transmits notices through a standard Bluetooth terminal application on an Android device; the central microcontroller parses the stream, commits it to memory, and drives a multiplexed LED matrix display.

---

## Features

- **Wireless Connectivity:** Seamless message transmission using the HC-05 Bluetooth module.
- **Real-Time Display Updates:** Immediate display refresh upon receiving a new payload over UART.
- **Dynamic Text Scrolling:** Smooth horizontal scrolling implementation across LED matrix columns.
- **Non-Volatile Storage:** Persists active notices in an external AT24C256 EEPROM via the I2C bus.
- **Power-Loss Recovery:** Automatically restores and scrolls the last-saved message upon system reboot or unexpected power loss.
- **Optimized Pin Usage:** Cascaded 74HC164 shift registers and 74HC573 latches minimize microcontroller GPIO overhead.

---

## Hardware Components

| Component | Description / Function | Quantity |
| :--- | :--- | :--- |
| **Microcontroller** | NXP LPC2129 (ARM7TDMI-S core, 16/32-bit) | 1 |
| **Bluetooth Module** | HC-05 (Serial SPP profile, UART interface) | 1 |
| **EEPROM** | AT24C256 (256 Kbit / 32 KB I2C EEPROM) | 1 |
| **Display Panel** | 8×8 LED Dot Matrix units | 4+ |
| **Shift Registers** | 74HC164 (8-bit Serial-In / Parallel-Out) | Modular |
| **Octal Latch** | 74HC573 (3-state non-inverting transparent latch) | 1 |
| **Power Supply** | Regulated 5V / 3.3V DC power source | 1 |
| **Passives & Misc** | Pull-up resistors (for I2C bus), 12 MHz crystal oscillator, decoupling capacitors | As needed |

---

## Software & Tools

- **Programming Language:** Embedded C
- **IDE & Compiler:** Keil µVision (ARM Compiler)
- **Flashing Software:** Flash Magic (In-System Programming over ISP/UART0)
- **Mobile Application:** Serial Bluetooth Terminal (Android) or any standard Bluetooth SPP terminal app

---

## System Architecture & Working Principle

```
+---------------------+           UART
|  Android Smartphone |  ======> [HC-05]
| (Bluetooth Terminal)|  (SPP)     |
+---------------------+            | Rx / Tx
                                   v
+------------------+   I2C Bus   +--------------------+
| AT24C256 EEPROM  | <=========> | LPC2129 Microcntr  |
+------------------+ (Save/Load) +--------------------+
                                   |
                                   | GPIO / Control
                                   v
                         +--------------------+
                         | 74HC164 & 74HC573  |
                         | (Drivers & Latches)|
                         +--------------------+
                                   |
                                   v
                         +--------------------+
                         | 8x8 LED Dot Matrix |
                         | (Scrolling Text)   |
                         +--------------------+
```

1. **Reception:** The user sends a string from a smartphone terminal. The **HC-05** module intercepts the Bluetooth packet and outputs raw bytes to the LPC2129 via **UART**.
2. **Buffering & Storage:** The microcontroller buffers incoming ASCII characters and commits the string into the **AT24C256 EEPROM** over the **I2C** interface, ensuring persistence across power interruptions.
3. **Display Multiplexing:**
   - The microcontroller extracts character font bitmasks from a lookup table.
   - Bit data is shifted serially into the **74HC164** shift register to drive display columns.
   - The **74HC573** latch controls row scanning to manage ghosting and ensure optimal refresh rates.
4. **Boot Sequence:** On power-up, the firmware queries the EEPROM address pointer. If a previously saved notice exists, it loads it directly into the active display buffer.

---


## Getting Started

### Prerequisites
1. Install **Keil µVision** with the legacy ARM7 device family pack installed.
2. Install **Flash Magic**.
3. Install any standard serial Bluetooth terminal on your mobile device (e.g., *Serial Bluetooth Terminal* from the Google Play Store).

### Flashing the Microcontroller
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/BLUETOOTH-BASED-DIGITAL-NOTICE-BOARDS.git
   ```
2. Open the project file in Keil µVision:
   ```plaintext
   firmware/project.uvproj
   ```
3. Build the target to generate the `.hex` binary file.
4. Put the LPC2129 board into **ISP Mode** (pull `P0.14` LOW and pulse reset).
5. Open Flash Magic:
   - Select device: **LPC2129**
   - Choose the active COM Port and set baud rate (typically `9600` or `19200`)
   - Select the generated `.hex` file
   - Click **Start** to burn the firmware.

---

## Usage

1. Power on the digital notice board hardware.
2. Turn on Bluetooth on your smartphone and pair with **HC-05** (Default PIN: `1234` or `0000`).
3. Launch your Bluetooth terminal app and connect to the module.
4. Type your message and send it:
   ```text
   ANN: Seminar postponed to 3:00 PM in Hall A#
   ```
5. The display will immediately update and begin scrolling your message continuously.
