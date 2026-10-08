# STM32F407 Vehicle Platform 🚗

### Bare-Metal Embedded Vehicle Controller using ARM Cortex-M4

A register-level embedded vehicle platform developed using the **STM32F407** microcontroller. The project demonstrates low-level peripheral programming, real-time motor control, Bluetooth communication, sensor interfacing, PWM generation, interrupt handling, and task scheduling **without using the STM32 HAL library**.

The platform is designed as a small-scale representation of an automotive embedded control system, where the STM32F407 acts as the central vehicle controller.

---

## 📌 Project Overview

The **STM32F407 Vehicle Platform** integrates multiple embedded peripherals to control and monitor a two-wheel DC motor vehicle.

The system receives commands through an **HC-05 Bluetooth module**, processes them using the STM32F407, and controls the motors through an **L298N motor driver**.

Additional features include:

- PWM-based motor speed control
- Forward/reverse/left/right movement
- Smooth speed ramping and deceleration
- Automatic headlight control using an LDR
- Accelerometer interfacing through SPI
- 20×4 LCD interfacing through I²C
- Periodic task scheduling
- Interrupt-driven peripheral handling
- Register-level peripheral configuration

---

# 🎯 Objectives

- Develop an embedded vehicle controller using the **STM32F407 Cortex-M4**.
- Implement peripherals directly through STM32 registers.
- Control DC motors using **PWM and GPIO**.
- Establish wireless communication using **USART and HC-05 Bluetooth**.
- Read analog light intensity using **ADC**.
- Interface an accelerometer using **SPI**.
- Interface a 20×4 LCD using **I²C**.
- Implement interrupt-based peripheral handling using **NVIC**.
- Design a lightweight periodic task scheduler.
- Implement safe vehicle speed transitions and direction changes.
- Gain practical experience with automotive-oriented embedded control systems.

---

# 🧠 System Architecture

```text
                         ┌─────────────────────────┐
                         │       STM32F407         │
                         │      ARM Cortex-M4       │
                         │                         │
                         │   Bare-Metal Firmware   │
                         └────────────┬────────────┘
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          │                           │                           │
          ▼                           ▼                           ▼
     ┌─────────┐                 ┌─────────┐                 ┌─────────┐
     │ USART2  │                 │  TIM2   │                 │  ADC1   │
     │         │                 │  PWM    │                 │         │
     └────┬────┘                 └────┬────┘                 └────┬────┘
          │                           │                           │
          ▼                           ▼                           ▼
     ┌─────────┐                ┌──────────┐                 ┌───────┐
     │  HC-05  │                │  L298N   │                 │  LDR  │
     │Bluetooth│                │ Motor    │                 └───────┘
     └─────────┘                │ Driver   │
                                └────┬─────┘
                                     │
                              ┌──────┴──────┐
                              ▼             ▼
                         ┌─────────┐   ┌─────────┐
                         │ Motor 1 │   │ Motor 2 │
                         └─────────┘   └─────────┘

          ┌───────────────────────────┼───────────────────────────┐
          │                           │
          ▼                           ▼
     ┌─────────┐                 ┌──────────┐
     │  SPI1   │                 │   I²C    │
     └────┬────┘                 └────┬─────┘
          │                           │
          ▼                           ▼
     ┌──────────┐                ┌──────────┐
     │ LIS302DL │                │ 20×4 LCD │
     │Accelerom.│                └──────────┘
     └──────────┘
```

---

# ⚙️ Hardware Components

| Component | Purpose |
|---|---|
| **STM32F407** | Main vehicle controller |
| **HC-05 Bluetooth** | Wireless command communication |
| **L298N Motor Driver** | DC motor control |
| **2 × DC Motors** | Vehicle movement |
| **LDR** | Ambient light detection |
| **LIS302DL Accelerometer** | Motion/acceleration sensing |
| **20×4 I²C LCD** | Status/data display |
| **LEDs** | Status indication |
| Battery/Power Supply | System power |

---

# 🔌 Peripheral Configuration

| STM32 Peripheral | Function |
|---|---|
| **RCC** | Peripheral clock configuration |
| **GPIO** | Digital I/O and motor direction control |
| **TIM2** | PWM generation and motor speed control |
| **USART2** | HC-05 Bluetooth communication |
| **ADC1** | LDR analog signal measurement |
| **SPI1** | LIS302DL accelerometer communication |
| **I²C** | 20×4 LCD communication |
| **NVIC** | Interrupt configuration |
| **SysTick/Timer-based scheduling** | Periodic task execution |

---

# 🚗 Vehicle Control

The vehicle supports basic motion commands:

```text
             FORWARD
                ↑
                │
      LEFT  ←  VEHICLE  →  RIGHT
                │
                ↓
             REVERSE
```

The Bluetooth module receives commands from a mobile device.

The STM32F407 processes the received command and generates the appropriate motor control signals.

### Example command flow

```text
Mobile Phone
     │
     ▼
   HC-05
     │
     │ UART
     ▼
  USART2
     │
     ▼
Command Parser
     │
     ▼
Vehicle Control
     │
     ▼
TIM2 PWM + GPIO
     │
     ▼
L298N
     │
     ▼
DC Motors
```

---

# ⚡ PWM Motor Control

**TIM2** is used to generate PWM signals for motor speed control.

The duty cycle determines the effective motor speed.

The firmware also implements **controlled speed transitions** instead of abruptly changing the motor output.

This allows the vehicle to:

- Accelerate smoothly
- Decelerate safely
- Stop before changing direction
- Reduce sudden mechanical stress on the motors

---

# 🔄 Safe Direction Change

The vehicle does not directly switch from forward to reverse at full speed.

Instead, the control logic follows:

```text
Forward
   │
   ▼
Reduce PWM
   │
   ▼
Motor Speed = 0
   │
   ▼
Change Direction
   │
   ▼
Increase PWM
   │
   ▼
Reverse
```

This demonstrates an important embedded control principle:

> **The software should enforce safe state transitions instead of blindly executing commands.**

---

# 📡 Bluetooth Communication

The **HC-05 Bluetooth module** communicates with the STM32F407 through **USART2**.

### Communication configuration

```text
Baud Rate : 9600
Protocol  : UART
Interface : USART2
```

The received command is processed by the firmware and mapped to a vehicle action.

```text
Received Command
       ↓
UART Receive
       ↓
Command Parser
       ↓
Vehicle State
       ↓
Motor Control
```

---

# 💡 Automatic Headlight Control

An **LDR** is connected to the STM32F407's ADC input.

The ADC converts the analog voltage from the LDR into a digital value.

```text
Ambient Light
      │
      ▼
     LDR
      │
      ▼
     ADC
      │
      ▼
Digital Value
      │
      ▼
Threshold Comparison
      │
 ┌────┴────┐
 ▼         ▼
Dark      Bright
 │           │
 ▼           ▼
Headlights  Headlights
   ON          OFF
```

This demonstrates practical use of **ADC-based sensor decision making**.

---

# 📐 Accelerometer Interface

The **LIS302DL accelerometer** is interfaced with the STM32F407 using **SPI1**.

```text
STM32F407
   │
   │ SPI1
   │
   ▼
LIS302DL
   │
   ▼
Acceleration Data
```

This demonstrates register-level implementation of the **SPI communication protocol**.

---

# 🖥️ LCD Interface

A **20×4 LCD with an I²C interface** is used to display vehicle information.

Example display:

```text
--------------------
 Vehicle Platform
--------------------
Speed : 60%
State : FORWARD
Light : ON
--------------------
```

The LCD communication is implemented through the STM32 I²C peripheral.

---

# ⏱️ Periodic Task Scheduler

The firmware uses a lightweight periodic scheduler to execute different tasks at defined intervals.

Example task periods:

| Task | Period |
|---|---:|
| Motor control | 10 ms |
| Vehicle state update | 50 ms |
| Sensor processing | 100 ms |
| Display update | 500 ms |

```text
                Scheduler
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    10 ms         50 ms        100 ms
       │            │            │
 Motor Control   State       Sensors
                 Update
                    │
                    ▼
                  500 ms
                    │
                    ▼
                LCD Update
```

---

# 🧩 Firmware Architecture

```text
                    main.c
                       │
                       ▼
              Application Layer
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Vehicle Control       Scheduler
             │                   │
             └─────────┬─────────┘
                       ▼
                Peripheral Layer
                       │
     ┌─────────┬───────┼───────┬─────────┐
     ▼         ▼       ▼       ▼         ▼
    GPIO      TIM     UART    ADC       SPI

                       │
                       ▼
                     I²C
                       │
                       ▼
                STM32F407 Registers
```

---

# 📁 Project Structure

```text
STM32F407-Vehicle-Platform/
│
├── Core/
│   ├── Inc/
│   └── Src/
│
├── Drivers/
│   ├── gpio/
│   ├── rcc/
│   ├── tim/
│   ├── usart/
│   ├── adc/
│   ├── spi/
│   └── i2c/
│
├── Application/
│   ├── vehicle_control.c
│   ├── scheduler.c
│   └── command_parser.c
│
├── Startup/
│
├── Documentation/
│   └── block_diagram.png
│
├── README.md
└── LICENSE
```

> Update the folder names above to match the actual structure of your repository.

---

# 🛠️ Technologies Used

### Hardware

- STM32F407
- ARM Cortex-M4
- HC-05 Bluetooth
- L298N Motor Driver
- DC Motors
- LDR
- LIS302DL Accelerometer
- 20×4 I²C LCD

### Software

- Embedded C
- STM32F407 Register-Level Programming
- ARM Cortex-M Architecture
- GPIO
- RCC
- Timers
- PWM
- USART
- ADC
- SPI
- I²C
- NVIC
- Interrupts

### Development

- STM32CubeIDE / compatible ARM GCC toolchain
- ST-LINK
- Git & GitHub

---

# 🚫 No STM32 HAL

A key objective of this project is understanding the STM32 peripheral architecture at the register level.

Peripheral initialization and control are implemented by configuring the corresponding STM32 registers rather than relying on STM32 HAL APIs.

This approach provides a deeper understanding of:

- Memory-mapped peripheral registers
- Clock trees
- GPIO alternate functions
- Timer operation
- Interrupt configuration
- Communication peripherals
- Embedded hardware/software interaction

---

# 🧪 Testing

The system was tested for:

- Bluetooth command reception
- Forward/reverse movement
- Left/right movement
- Motor stopping
- PWM speed control
- Smooth acceleration/deceleration
- Direction transition
- LDR-based headlight control
- SPI accelerometer communication
- I²C LCD communication
- Periodic task execution
- Interrupt-based peripheral operation

---

# 📊 Key Embedded Concepts Demonstrated

- ARM Cortex-M4 architecture
- Bare-metal Embedded C
- Register-level programming
- RCC clock configuration
- GPIO configuration
- Timer/PWM generation
- UART/USART communication
- ADC
- SPI
- I²C
- NVIC and interrupts
- Periodic task scheduling
- State-based vehicle control
- Motor control
- Sensor interfacing
- Real-time embedded programming
- Hardware/software debugging

---

# 🚘 Automotive Relevance

Although this is a small-scale vehicle platform, its architecture demonstrates concepts used in automotive embedded systems:

```text
Sensor Input
     ↓
Signal Processing
     ↓
Decision Making
     ↓
Control Logic
     ↓
Actuator Output
```

The project provides hands-on exposure to:

- Automotive ECUs
- Body control systems
- Vehicle control units
- Sensor interfaces
- Motor control systems
- Embedded communication systems

---

# 📚 References

- **STM32F407 Reference Manual – RM0090**
- **STM32F407 Datasheet**
- **ARM Cortex-M4 Technical Documentation**
- STM32F407 device programming documentation

---

# 👩‍💻 Author

**Bhavna R**

Electronics & Communication Engineering  
Embedded Systems | ARM | C/C++ | Automotive Electronics

GitHub:  
https://github.com/bhavna3030

LinkedIn:  
https://www.linkedin.com/in/bhavna-raj

---

# ⭐ Project Highlights

```text
ARM Cortex-M4
      +
Bare-Metal C
      +
Register-Level Programming
      +
PWM Motor Control
      +
UART Bluetooth
      +
ADC Sensor
      +
SPI Accelerometer
      +
I²C LCD
      +
Interrupts
      +
Real-Time Scheduling
      ↓
STM32F407 Vehicle Platform
```

---

## 📌 License

This project is intended for educational and portfolio purposes.
