# STM32 CAN Communication Test

This repository demonstrates a simple two-node CAN communication setup using STM32 microcontrollers. It is designed to validate CAN bus communication between two STM32G474RE NUCLEO development boards connected through TJA1051 CAN transceivers.

## Hardware Used

* **STM32G474RE NUCLEO** development boards (2)
* **CJMCU TJA1051** CAN transceiver modules (2)
* **STM32CubeIDE**
* **STM32 HAL** drivers

## Repository Structure

```text
├── Node_A_TX/     # CAN transmitter firmware
├── Node_B_RX/     # CAN receiver firmware
├── Common/        # Shared CAN definitions and configuration
└── Docs/          # Wiring diagrams, configuration details, and test documentation
```

## Project Status

* ✅ Node A (Transmitter) configured
* ✅ Node B (Receiver) configured
* ✅ CAN peripheral initialization completed
* ⏳ Ready for hardware validation and communication testing
