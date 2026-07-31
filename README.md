# STM32 CAN Communication Test

A bring-up and validation project for establishing **Classic CAN communication** between two **STM32G474RE NUCLEO** development boards using **CJMCU TJA1051 CAN transceivers**.

The primary objective of this project is to verify the hardware, understand the STM32 FDCAN peripheral, and build a reliable foundation before scaling to larger CAN networks.

---

## Objectives

- Verify the functionality of the CJMCU TJA1051 CAN transceiver.
- Establish communication between two STM32G474RE nodes.
- Configure the STM32 FDCAN peripheral using STM32CubeIDE.
- Understand CAN filters, message transmission, and reception.
- Document the complete debugging process.
- Build a reusable test setup for future embedded and CubeSat projects.

---

## Hardware Used

| Component | Quantity |
|-----------|---------:|
| STM32 NUCLEO-G474RE | 2 |
| CJMCU TJA1051 CAN Transceiver | 2 |
| Breadboards | 2 |
| Jumper Wires | As required |

---

## Software Used

- STM32CubeIDE 1.9.0
- STM32 HAL Drivers
- Git
- GitHub

---

## Repository Structure

```text
STM32_CAN_Test/
│
├── Common/
│   ├── can_ids_test.h
│   └── can_messages_test.h
│
├── Docs/
│   ├── CAN_Test_Procedure.md
│   ├── Debugging_Report.md
│   ├── Troubleshooting.md
│   ├── Wiring.md
│   └── screenshots/
│
├── Node_A_TX/
│
├── Node_B_RX/
│
└── README.md
```

---

## Project Workflow

```
Node A (STM32)
        │
        ▼
FDCAN Peripheral
        │
        ▼
CJMCU TJA1051
        │
──────── CAN Bus ────────
        │
CJMCU TJA1051
        │
        ▼
FDCAN Peripheral
        │
        ▼
Node B (STM32)
```

---

## Documentation

Detailed documentation is available in the **Docs** folder.

| Document | Description |
|----------|-------------|
| Wiring.md | Hardware connections |
| CAN_Test_Procedure.md | Step-by-step procedure to reproduce the setup |
| Debugging_Report.md | Complete debugging journey from failure to successful communication |
| Troubleshooting.md | Common CAN issues and their solutions |

---

## Current Status

- ✅ Two-node CAN communication established
- ✅ CAN transmission verified
- ✅ CAN reception verified
- ✅ Live Expressions used for debugging
- ✅ Hardware wiring validated

---

## Future Work

- Bidirectional communication
- Silent Mode testing
- Loopback Mode testing
- CAN Filters
- Remote Frames
- Extended Identifiers
- Interrupt-based reception
- CAN FD
- Multi-node CAN network