# CAN Test Procedure

This document describes the procedure used to verify CAN communication between two STM32G474RE NUCLEO boards using CJMCU TJA1051 CAN transceivers.

---

# Objective

Verify successful Classic CAN communication between two STM32G474RE nodes.

- Node A → Transmitter
- Node B → Receiver

---

# Hardware Required

| Component | Quantity |
|-----------|---------:|
| STM32 NUCLEO-G474RE | 2 |
| CJMCU TJA1051 | 2 |
| Breadboard | 2 |
| Jumper Wires | As required |

---

# Software Required

- STM32CubeIDE 1.9.0
- STM32 HAL Drivers
- Git
- ST-Link Debugger

---

# Step 1 – Flash the Firmware

Program the following projects:

| Board | Firmware |
|--------|----------|
| Node A | Node_A_TX |
| Node B | Node_B_RX |

Ensure both projects build successfully without errors or warnings.

---

# Step 2 – Verify Hardware Connections

Before powering the boards, verify:

- [ ] PA12 → TXD
- [ ] PA11 → RXD
- [ ] 3.3V → VCC
- [ ] GND → GND
- [ ] CANH ↔ CANH
- [ ] CANL ↔ CANL


Refer to **Wiring.md** for the complete connection diagram.

---

# Step 3 – Start Debugging

Debug both projects using STM32CubeIDE.

Open the **Live Expressions** window for each node.

---

# Step 4 – Observe Node A (Transmitter)

Monitor the following variables:

```c
txStatus
txLevel
fdcanError
protocolStatus
```

Expected result:

| Variable | Expected Value |
|-----------|----------------|
| txStatus | HAL_OK |
| txLevel | Stable (does not continuously decrease to 0) |
| fdcanError | 0 |
| BusOff | 0 |
| ErrorPassive | 0 |
| Warning | 0 |

---

# Step 5 – Observe Node B (Receiver)

Monitor:

```c
rxStatus
rxCount
messageReceived
RxHeader.Identifier
RxData
```

Expected result:

| Variable | Expected Value |
|-----------|----------------|
| rxStatus | HAL_OK |
| rxCount | Increases continuously |
| messageReceived | 1 |
| RxHeader.Identifier | 0x123 |
| RxData | Matches transmitted data |

---

# Step 6 – Verify Communication

Communication is successful when:

- Node A continuously transmits frames.
- Node B continuously receives frames.
- `rxCount` increases.
- `txStatus` remains `HAL_OK`.
- `fdcanError` remains `0`.
- No Bus-Off condition occurs.

---

# Expected Live Expressions

## Node A

![<p align="center">
<img src="screenshots/08_successful_tx.png" width="700">
</p>](file:///c%3A/Users/devan/OneDrive/Desktop/SSP/CAN%20TEST%20IMG/Screenshot%202026-07-31%20192823.png)

**Figure 1.** Successful transmitter status.

---

## Node B

![<p align="center">
<img src="screenshots/09_successful_rx.png" width="700">
</p>
](file:///c%3A/Users/devan/OneDrive/Desktop/SSP/CAN%20TEST%20IMG/RX%20final%20LE.jpeg)
**Figure 2.** Successful receiver status.

---

# Validation Checklist

| Check | Status |
|-------|:------:|
| Firmware compiled successfully | ✅ |
| Both boards powered | ✅ |
| FDCAN started | ✅ |
| CAN frames transmitted | ✅ |
| CAN frames received | ✅ |
| rxCount increasing | ✅ |
| No HAL errors | ✅ |
| No Bus-Off | ✅ |

---

# Test Result

The CAN communication test was completed successfully.

- Node A transmitted Classic CAN frames.
- Node B received the transmitted frames.
- The CJMCU TJA1051 transceivers operated correctly.
- The STM32G474RE FDCAN peripheral functioned as expected.

This validates both the hardware setup and the software configuration for a two-node CAN network.