# STM32 FDCAN Two-Node Communication Debugging Report

## Objective

The objective of this project was to establish Classic CAN communication between two STM32G474RE NUCLEO development boards using CJMCU TJA1051 CAN transceiver modules.

The initial goal was to verify the functionality of the CAN transceivers by implementing one-way communication:

- **Node A** – Transmitter
- **Node B** – Receiver

Communication was monitored using **STM32CubeIDE Live Expressions**, without UART logging.

---

# Hardware Configuration

| Component | Quantity |
|-----------|---------:|
| STM32 NUCLEO-G474RE | 2 |
| CJMCU TJA1051 CAN Transceiver | 2 |
| Breadboards | 2 |
| Jumper Wires | As required |

---

# Software Environment

| Tool | Version |
|------|---------|
| STM32CubeIDE | 1.9.0 |
| STM32 HAL Drivers | STM32G4 HAL |
| Protocol | Classic CAN |

---

# Initial Problem

The firmware compiled successfully on both nodes and the FDCAN peripheral initialized without errors. However, CAN communication could not be established.

Observed behaviour:

- Node A continuously attempted to transmit frames.
- Node B never received any CAN frames.
- Communication failed despite successful initialization.

---

# Debugging Timeline

## Investigation 1 – Peripheral Initialization

### Objective

Verify that the FDCAN peripheral was initialized correctly.

### APIs Used

```c
HAL_FDCAN_Init()

HAL_FDCAN_Start()
```

### Result

```
HAL_OK
```

for both APIs.

### Conclusion

The peripheral initialized successfully and entered Normal mode. Initialization was ruled out as the cause of the problem.

---

## Investigation 2 – Transmit FIFO

### APIs Used

```c
HAL_FDCAN_AddMessageToTxFifoQ()

HAL_FDCAN_GetTxFifoFreeLevel()
```

### Observation

```
txLevel

3
↓

2
↓

1
↓

0
```

After the FIFO became full,

```
txStatus

HAL_ERROR
```

### Screenshot

![alt text](<Screenshot 2026-07-30 192547.png>)

### Conclusion

Frames were successfully queued into the transmit FIFO but were never removed after transmission.

---

## Investigation 3 – HAL Error Status

### API Used

```c
fdcanError = HAL_FDCAN_GetError(&hfdcan1);
```

### Observation

```
fdcanError = 512
```

which corresponds to

```
HAL_FDCAN_ERROR_FIFO_FULL
```

### Conclusion

The transmit FIFO became full because transmitted frames were never successfully removed from the controller.

---

## Investigation 4 – Protocol Status

### API Used

```c
HAL_FDCAN_GetProtocolStatus(&hfdcan1, &protocolStatus);
```

### Observation

Node A

```
BusOff = 1

ErrorPassive = 1

Warning = 1
```

### Screenshot

![alt text](<Screenshot 2026-07-31 165154.png>)

### Conclusion

Repeated communication failures caused the CAN controller to enter the Bus-Off state.

---

## Investigation 5 – Receiver Status

### API Used

```c
HAL_FDCAN_GetRxMessage()
```

### Observation

```
rxStatus = HAL_ERROR

rxCount = 0
```

### Screenshot

![alt text](<Screenshot 2026-07-31 170458.png>)

### Conclusion

The receiver never received any CAN frame.

---

## Investigation 6 – Software Verification

The following parameters were verified:

- HAL_FDCAN_Init()
- HAL_FDCAN_Start()
- CAN Filter configuration
- Standard Identifier
- CAN Bit Timing
- FDCAN Mode
- Auto Retransmission

Both Node A and Node B used identical bit timing:

| Parameter | Value |
|-----------|------:|
| Prescaler | 20 |
| SJW | 2 |
| TimeSeg1 | 14 |
| TimeSeg2 | 2 |

No software configuration issues were found.

---

## Investigation 7 – Hardware Inspection

After eliminating software-related causes, the hardware connections were inspected.

The STM32 FDCAN pins were found to be connected to incorrect NUCLEO header pins.

As a result:

- No CAN frames reached the transceiver.
- Node A entered Bus-Off.
- Node B never received any frames.

### STM32 Pinout

![alt text](Nucleo-G474RE_PinOut-1.png)
### Corrected Wiring

![alt text](<wring set up-1.jpeg>)

After correcting the STM32 pin mapping, communication was established immediately.

---

# Verification After Fix

The following observations confirmed successful communication.

## Node A

- txStatus = HAL_OK
- fdcanError = 0
- BusOff = 0

<p align="center">
<img src=![alt text](<TX final LE.png>)>
</p>

---

## Node B

- rxStatus = HAL_OK
- rxCount increasing
- messageReceived = 1

![
    
](<RX final LE.jpeg>)

Communication was successfully established between both STM32 nodes.

---

# Root Cause

The root cause of the communication failure was incorrect MCU pin mapping.

Although the STM32 FDCAN peripheral was correctly configured and initialized, the PA12 (FDCAN1_TX) and PA11 (FDCAN1_RX) signals were connected to incorrect NUCLEO header pins. Consequently, the CJMCU TJA1051 transceiver never received valid transmit and receive signals, preventing communication on the CAN bus.

Correcting the hardware wiring resolved the issue without requiring any software modifications.

---

# Lessons Learned

- Successful peripheral initialization does not guarantee successful CAN communication.
- A full transmit FIFO is generally a symptom of a lower-level communication problem.
- HAL_FDCAN_GetError() is useful for identifying HAL-level issues.
- HAL_FDCAN_GetProtocolStatus() provides valuable insight into the CAN controller state.
- Debugging should eliminate software causes before modifying hardware.
- Always verify MCU pin mapping using the official board documentation before investigating higher software layers.

---

# Troubleshooting Summary

| Symptom | Diagnostic Tool | Observation | Conclusion |
|---------|-----------------|-------------|------------|
| txStatus = HAL_ERROR | HAL_FDCAN_AddMessageToTxFifoQ() | FIFO became full | Frames not leaving TX FIFO |
| fdcanError = 512 | HAL_FDCAN_GetError() | HAL_FDCAN_ERROR_FIFO_FULL | FIFO full due to failed communication |
| BusOff = 1 | HAL_FDCAN_GetProtocolStatus() | Controller stopped transmitting | Repeated communication failures |
| rxStatus = HAL_ERROR | HAL_FDCAN_GetRxMessage() | No received frames | Receiver not receiving CAN traffic |
| Communication restored | Physical inspection | Corrected MCU pin mapping | Root cause identified and resolved |

---

# Final Result

The STM32G474RE FDCAN peripheral and CJMCU TJA1051 transceivers were successfully validated through two-node Classic CAN communication.

The project established a reliable hardware and software foundation for future work involving:

- Bidirectional CAN communication
- Silent Mode
- Loopback Mode
- CAN Filters
- Extended Identifiers
- Interrupt-based reception
- CAN FD
- Multi-node CAN networks