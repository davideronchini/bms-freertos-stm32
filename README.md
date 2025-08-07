# BMS - FreeRTOS 
**`Nucleo F446RE ⚡ + FreeRTOS ⏱️`**

This repository contains the firmware for managing the Battery Management System (BMS) of the 2024–2025 Formula SAE electric vehicle, running on the **STM32 Nucleo-F446RE** with **FreeRTOS**.  
It includes voltage and temperature monitoring using **LTC6811-1** battery monitoring ICs via **isoSPI (LTC6820)**, fault detection logic, CAN bus communication, and safe control of high-voltage contactors.

---

## ✅ Features

- 🔋 **Cell Monitoring with LTC6811**  
  Monitors voltage and temperature using Analog Devices’ LTC6811-1 with high accuracy over isoSPI via LTC6820.
  
- 🌡️ **Temperature Sensing**  
  Battery pack thermal protection based on real-time cell temperature values.

- ⚠️ **Fault Detection & Contactor Control**  
  Handles overvoltage, undervoltage, overtemperature, and insulation faults. Automatically disconnects HV through contactor logic.

- 🛠️ **Modular FreeRTOS Architecture**  
  Uses multiple preemptive tasks for safety, communication, estimation and system diagnostics.

- 🛞 **CAN Bus Communication**  
  Transmits telemetry and system status to dashboard, VCU or data logger. Receives control commands.

- 🔌 **Charging Management**  
  Detects charger plug-in state and manages the charging session safely.

- 🧠 **SOC Estimation Task (optional)**  
  Designed to integrate a Kalman Filter or Coulomb counter for State of Charge estimation.

---

## 🧠 Architecture Overview

The firmware is organized in a modular structure to support testability, scalability, and clarity:

```
/Core
├── /Inc
│   ├── main.h
│   ├── freertos_config.h
│   └── system_state.h
├── /Src
│   ├── main.c
│   ├── freertos_config.c
│   └── system_state.c

/Application
├── cell_monitoring.c/.h
├── fault_handler.c/.h
├── soc_estimation.c/.h
├── charging_control.c/.h
├── can_tx.c/.h
└── contactor_control.c/.h

/Drivers
├── /AnalogDevices_LTC
│   ├── ltc6811.c/.h
│   ├── ltc6811_reg.h
│   ├── ltc681x.c/.h
│   ├── ltc6820.c/.h
│   └── spi_hal.c/.h
└── /BMS_HAL
    ├── bms_adc.c/.h
    ├── bms_can.c/.h
    ├── bms_gpio.c/.h
    └── bms_debug.c/.h

/Middleware
└── /FreeRTOS

/Config
├── FreeRTOSConfig.h
└── system_config.h
```

---

## 🔄 Scheduler & Task Management (FreeRTOS)

The system uses FreeRTOS to separate safety-critical and non-critical tasks. Tasks include:

| Task Name             | Functionality                             | Priority |
|-----------------------|-------------------------------------------|----------|
| `Task_FaultHandler`   | Evaluate safety flags and disable HV      | 5        |
| `Task_CellMonitoring` | Read LTC6811 voltage/temperature          | 4        |
| `Task_CANTransmit`    | Send telemetry on CAN                     | 3        |
| `Task_CANReceive`     | Process incoming CAN messages             | 3        |
| `Task_SOC_Estimation` | Estimate State of Charge                  | 3        |
| `Task_ChargingControl`| Manage charging session                   | 3        |
| `Task_USBLogger`      | Send debug data over UART                 | 1        |
| `Task_Watchdog`       | Reset the board on fault                  | 2        |

### Semaphores & Queues

| Name                | Type               | Purpose                                   |
|---------------------|--------------------|-------------------------------------------|
| `Queue_CellData`    | Queue              | Pass measured cell data between tasks     |
| `Queue_Faults`      | Queue              | Deliver fault events to FaultHandler      |
| `BinarySemaphore_IMD`| Binary Semaphore  | Signals insulation fault from ISR         |
| `Mutex_BMSData`     | Mutex              | Protect access to shared BMS structs      |

---

## 📡 Communication

### CAN Bus

- Configured at **500 kbps**
- Used for:
  - Sending periodic status (voltage, temp, SOC, faults)
  - Receiving remote commands (shutdown, charge, etc.)
- CAN HAL integrated into `bms_can.c`

### Serial Debug (UART2)

- Transmits logs, SOC, and fault flags over USB
- Useful for terminal tools like **Termite** or **SerialTools**

---

## 🔌 Battery Monitoring with LTC6811

- The project uses **LTC6811-1** and **LTC6820** connected via **isoSPI**
- Communication via **SPI1**
- Functions provided by Analog Devices are wrapped in `Drivers/AnalogDevices_LTC`
- Interface managed by `cell_monitoring.c` task

### DWT Delay

Microsecond-level delays (e.g. for LTC commands) are implemented with `DWT_Delay.h`, based on the ARM Core debug timer:

```c
DWT_Delay_Init();         // Call once in system init
DWT_Delay_us(200);        // Delay 200 microseconds
```

---

## 🛠️ Getting Started

To run the BMS firmware:

1. **Clone the repository**

   ```bash
   git clone https://github.com/PolimarcheRacingTeam/BMS-24-25.git
   ```

2. **Open in STM32CubeIDE**

3. **Compile and upload to STM32 Nucleo-F446RE**

4. **Connect CAN, isoSPI (LTC6820 → LTC6811), UART**

5. **Use serial monitor to check debug messages**

6. **Use CAN tools (e.g. PCAN, Kvaser, or CANable) to monitor packets**

---

## 👨‍🔧 Advanced Configuration

- Stack sizes for each task are monitored via `uxTaskGetStackHighWaterMark()`
- Memory allocation uses **heap_4.c** (fragmentation-safe, supports `vPortFree()`)
- You can define custom faults and error IDs in `fault_handler.h`

---

## 📚 Credits & Resources

- **Analog Devices LTC6811**  
   [Product Page](https://www.analog.com/en/products/ltc6811-1.html)

- **STM32 Nucleo-F446RE**  
   📄 [Board Overview](https://os.mbed.com/platforms/ST-Nucleo-F446RE/)

- **CAN Bus Overview**  
   🎞️ [CAN Bus Tutorial](https://www.youtube.com/watch?v=KHNRftBa1Vc)

- **FreeRTOS**  
   📘 [FreeRTOS Official Documentation](https://freertos.org/Documentation/RTOS_book.html)

---

## 📬 Contact

For questions, contributions, or collaborations with my FS Team:

- 📧 Email: [formulasae@sm.univpm.it](mailto:formulasae@sm.univpm.it)
- 🔗 LinkedIn: [polimarcheracingteam](https://www.linkedin.com/company/polimarcheracingteam/posts/?feedView=all)
- 📸 Instagram: [@polimarcheracingteam](https://www.instagram.com/polimarcheracingteam/)

---
