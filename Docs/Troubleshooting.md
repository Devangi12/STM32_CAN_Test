# STM32 FDCAN Troubleshooting Guide

This document summarizes common issues encountered during the development of this project, the symptoms observed, diagnostic methods used, and their corresponding solutions.

---

# Quick Diagnostic Flow

```
Communication Failure
        │
        ▼
Is HAL_FDCAN_Start() successful?
        │
        ├── No → Check FDCAN initialization
        │
        ▼
Is txStatus == HAL_OK?
        │
        ├── No → Check HAL_FDCAN_GetError()
        │
        ▼
Read Protocol Status
        │
        ▼
Check Receiver
        │
        ▼
Verify Wiring
        │
        ▼
Communication Restored
```

---

# Common Issues

## 1. Transmit FIFO Full

### Symptoms

- `txStatus = HAL_ERROR`
- `txLevel` decreases continuously
- Communication stops

### Diagnostic API

```c
HAL_FDCAN_GetError(&hfdcan1);
```

### Result

```
fdcanError = 512
```

which corresponds to

```
HAL_FDCAN_ERROR_FIFO_FULL
```

### Possible Causes

- Receiver not acknowledging frames
- CAN bus disconnected
- Incorrect wiring
- Bus-Off state

### Solution

- Verify CANH/CANL connections
- Verify common ground
- Verify receiver is running
- Check protocol status

---

## 2. FIFO Empty

### Symptoms

Receiver continuously returns

```c
HAL_ERROR
```

### Diagnostic API

```c
HAL_FDCAN_GetError()
```

### Result

```
HAL_FDCAN_ERROR_FIFO_EMPTY
```

### Meaning

No message is available inside the receive FIFO.

### Possible Causes

- No transmitter
- Incorrect CAN Identifier
- Filter rejecting frames
- Wrong bit timing

---

## 3. Bus-Off

### Diagnostic API

```c
HAL_FDCAN_GetProtocolStatus()
```

### Symptoms

```
BusOff = 1
```

### Meaning

The CAN controller detected excessive communication errors and disconnected itself from the CAN bus.

### Possible Causes

- Incorrect wiring
- No ACK
- Disconnected CAN bus
- Wrong baud rate

### Solution

- Verify physical connections
- Verify both nodes use identical bit timing
- Restart the controller after fixing the issue

---

## 4. Error Passive

### Symptoms

```
ErrorPassive = 1
```

### Meaning

The controller has accumulated a high error count and is operating in Error Passive mode.

### Possible Causes

- Noise on the bus
- Missing termination
- Wiring issues

### Solution

Inspect the physical layer before investigating software.

---

## 5. Warning State

### Symptoms

```
Warning = 1
```

### Meaning

The transmit or receive error counter exceeded the warning threshold.

### Solution

Investigate communication errors before they develop into a Bus-Off condition.

---

## 6. Receiver Never Receives Frames

### Symptoms

```
rxStatus = HAL_ERROR

rxCount = 0

messageReceived = 0
```

### Diagnostic APIs

```c
HAL_FDCAN_GetRxMessage()

HAL_FDCAN_GetProtocolStatus()
```

### Possible Causes

- Incorrect filter
- Wrong Identifier
- Wrong wiring
- Transmitter not running

### Solution

- Verify filters
- Verify CAN IDs
- Verify wiring
- Verify transmitter status

---

## 7. Communication Fails Despite Successful Initialization

### Symptoms

```
HAL_FDCAN_Init()

HAL_FDCAN_Start()

↓

HAL_OK
```

but

```
No Communication
```

### Meaning

Peripheral initialization only confirms that the controller has been configured correctly.

It does **not** guarantee successful CAN communication.

### Possible Causes

- Wiring errors
- Missing termination resistors
- Wrong bit timing
- Incorrect pin mapping

---

## 8. Incorrect MCU Pin Mapping (Root Cause of This Project)

### Symptoms

- FIFO Full
- Bus-Off
- No received frames
- HAL_ERROR during transmission

### Root Cause

The STM32 FDCAN pins were connected to incorrect NUCLEO header pins.

Although the software configuration was correct, the CAN transceiver never received valid TX/RX signals.

### Solution

Correct the STM32 connections:

| STM32 | TJA1051 |
|--------|----------|
| PA12 | TXD |
| PA11 | RXD |
| 3.3V | VCC |
| GND | GND |

Communication was immediately restored after correcting the wiring.

---

# Useful HAL APIs

| API | Purpose |
|------|---------|
| `HAL_FDCAN_Init()` | Initialize the FDCAN peripheral |
| `HAL_FDCAN_Start()` | Start CAN communication |
| `HAL_FDCAN_AddMessageToTxFifoQ()` | Queue a message for transmission |
| `HAL_FDCAN_GetRxMessage()` | Read a received CAN frame |
| `HAL_FDCAN_GetError()` | Read HAL error flags |
| `HAL_FDCAN_GetProtocolStatus()` | Read controller status |
| `HAL_FDCAN_ConfigFilter()` | Configure acceptance filters |

---

# Live Expressions Used

## Node A

- txStatus
- txLevel
- fdcanError
- protocolStatus
- startStatus

---

## Node B

- rxStatus
- rxCount
- messageReceived
- RxHeader.Identifier
- RxData

---

# Best Practices

- Verify wiring before modifying software.
- Ensure both nodes use identical CAN bit timing.
- Always connect a common ground.
- Use 120 Ω termination resistors at both ends of the CAN bus if not present internally.
- Verify MCU pin mapping using the official board documentation.
- Read HAL error flags before assuming software faults.
- Monitor protocol status to identify Bus-Off and Error Passive conditions.

---

# Project Outcome

The debugging process demonstrated that software diagnostics alone are not always sufficient. By systematically verifying initialization, runtime status, error flags, protocol status, and hardware connections, the root cause was identified as incorrect MCU pin mapping.

After correcting the hardware wiring, stable CAN communication was successfully established between both STM32G474RE nodes.