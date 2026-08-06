# Camosun ROV — UART Test (Raspberry Pi ↔ STM32F446)

Firmware for testing UART communication between a Raspberry Pi and an
STM32F446 Nucleo board, used to remotely toggle onboard/external LEDs as a
sanity check before wiring up the full ROV control link.

## Overview

- The Pi sends packets over UART to the STM32 (UART4, 115200 8N1).
- The STM32 receives data via DMA using `HAL_UARTEx_ReceiveToIdle_DMA`, so
  packets of variable length are captured whenever the line goes idle
  (no fixed-length framing needed).
- On receipt, the main loop decodes the packet and sets LED states
  accordingly.
- USART2 is also initialized (115200 8N1), reserved for future use (e.g.
  ST-Link virtual COM port debugging).

## Packet Format

Each packet is a sequence of `(ledId, state)` byte pairs, so total length
must be even:

| Byte | Meaning |
|------|---------|
| 0    | LED ID   |
| 1    | State (`0x01` = ON, else OFF) |
| 2    | LED ID (next pair) |
| 3    | State |
| ...  | ... |

**LED IDs:**

| ID     | Pin              |
|--------|------------------|
| `0x01` | `LED_Pin` (PC8)  |
| `0x02` | `LD2_Pin` (PA5, onboard LED) |

Any other ID is ignored. Malformed (odd-length) packets are dropped.

Example: to turn LED 1 ON and LED 2 OFF in one packet, send:
`01 01 02 00`

## Hardware

- **Board:** STM32F446 (Nucleo)
- **UART4:** Pi ↔ STM32 communication, 115200 baud, 8N1, no flow control
- **USART2:** reserved / debug
- **LD2 (PA5):** onboard LED — toggles once per valid packet as a receive
  confirmation, then reflects its own state if targeted by ID `0x02`
- **LED (PC8):** external LED, controlled via ID `0x01`
- **B1 / BTN:** user button input (configured, not yet used in logic)
