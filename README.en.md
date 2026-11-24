# KinetiCoreDC

![MCU](https://img.shields.io/badge/MCU-STM32G070CBT6-03234B?style=flat-square&logo=stmicroelectronics)
![Driver](https://img.shields.io/badge/Driver-DRV8874PWPR-CC0000?style=flat-square&logo=texasinstruments)
![EDA](https://img.shields.io/badge/KiCad-10.0-314CB0?style=flat-square&logo=kicad)
![License](https://img.shields.io/badge/License-CERN--OHL--S--2.0-blue?style=flat-square)

🌐 **Language / Langue :** [🇫🇷 Français](README.md) | [🇬🇧 English](README.en.md)

---

**KinetiCoreDC** is a ruggedized closed-loop DC motor controller designed to turn any brushed DC motor into a **smart servomotor** (position, velocity, and torque control). Developed by **Paul Miailhe** for **Berry Rocket**.

© 2025/2026 - **Paul Miailhe** (*Berry Rocket*).  
This material is licensed under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S)**. $\rightarrow$ [LICENSE](LICENSE)

---

## 1. Overview & Applications

Built around an **STM32G070CBT6** microcontroller (ARM Cortex-M0+ @ $64\text{ MHz}$, internal HSI oscillator) and an integrated **DRV8874PWPR** H-bridge driver, the board combines all power, protection, and signal-conditioning electronics required for precise actuation in demanding embedded environments:

- **Multi-Axis Digital Servomotor:** Closed-loop PID control (position / velocity / torque) driven and monitored over a half-duplex RS-485 bus (`THVD1400D`, up to $500\text{ kbps}$).
- **Fail-Safe or Redundant Actuation:** PWM duty-cycle or RC pulse ($1\text{–}2\text{ ms}$) control via two opto-isolated inputs (`TLP2310`), supporting a secondary backup command channel.
- **Mechatronics & Mobile Robotics:** Brushed DC motor control with absolute (`AS5047P`, `AS5600`) or incremental Hall encoder feedback, hardware limit-switch homing, and current-based torque limiting (`IPROPI`).

---

## 2. Electrical Characteristics

The input power stage is designed for **2S or 3S LiPo** battery packs (or regulated DC supplies). An **LTC4365TS8** surge stopper driving back-to-back N-Channel MOSFETs (**SiS412DN**) provides hardware disconnect under undervoltage, overvoltage, or reverse-polarity conditions, while an **SMBJ15CA** TVS diode protects the $+3.3\text{ V}$ regulator against voltage transients.

| Parameter | Symbol | Min | Typ | Max | Unit | Conditions / Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Input Supply Voltage** | $V_{\text{IN}}$ | $5.90$ | $7.4\text{–}11.1$ | **$14.75$** | $\text{V}$ | Native 2S ($8.4\text{ V}$) and 3S ($12.6\text{ V}$) LiPo support |
| **Reverse Polarity Rating** | $V_{\text{REV}}$ | **$-40$** | — | — | $\text{V}$ | Blocked by back-to-back N-MOSFETs |
| **Undervoltage Lockout (fall)** | $V_{\text{UVLO}}$ | — | $5.90$ | — | $\text{V}$ | Turn-on threshold at $6.20\text{ V}$ (default config) |
| **Overvoltage Lockout (rise)** | $V_{\text{OVLO}}$ | — | **$14.75$** | — | $\text{V}$ | Recovery at $14.01\text{ V}$ (`SMBJ15CA` TVS clamp) |
| **Regulated Motor Current** | $I_{\text{TRIP}}$ | — | **$2.6$** | **$2.6$** | $\text{A}$ | $V_{\text{REF}} = 2.50\text{ V}$, $R_{\text{IPROPI}} = 2.49\text{ k}\Omega$ |
| **Peak Motor Current** | $I_{\text{PEAK}}$ | — | — | **$6.0$** | $\text{A}$ | `DRV8874` silicon limit (`XT30` rated $15\text{ A}$ cont.) |
| **Max Input Power** | $P_{\text{IN}}$ | — | $32.8$ | $38.4$ | $\text{W}$ | At $I_{\text{TRIP}} = 2.6\text{ A}$ (up to $88.5\text{ W}$ peak at $6\text{ A}$) |
| **Logic Supply Rail** | $V_{\text{3V3}}$ | $3.28$ | $3.30$ | $3.32$ | $\text{V}$ | $\pm 0.4\text{ \%}$ accuracy (`MCP1703A`, SOT-223 package) |
| **Available $+3.3\text{ V}$ Current** | $I_{\text{OUT(3V3)}}$ | — | — | **$250$** | $\text{mA}$ | Board draw ($\approx 50\text{ mA}$) + external sensors |
| **Shutdown Quiescent Current** | $I_{\text{SHDN}}$ | — | — | **$< 10$** | $\mu\text{A}$ | `SHDN_EXT` pin pulled to `GND` |

> **Threshold Customization:** The UVLO/OVLO window can be tailored to specific battery chemistries (dedicated 2S LiPo $6.45\text{–}10.84\text{ V}$, dedicated 3S LiPo $9.15\text{–}14.28\text{ V}$, or 9V battery) via the `LTC4365` resistor divider on `Power.kicad_sch`. Similarly, the current limit $I_{\text{TRIP}}$ can be adjusted up to $6\text{ A}$ via `R_IPROPI`.

---

## 3. Supported Interfaces & Sensors

All external I/Os feature dedicated protection against electrostatic discharge (ESD), miswiring, and $32\text{ kHz}$ PWM switching noise. External $+3.3\text{ V}$ sensor power rails (`J10` and `J12`) are current-limited via $22\ \Omega$ series resistors, while mechanical inputs are hardware-filtered and conditioned through Schmitt-trigger inverters (**SN74LVC2G14DBV**).

| Connector | Interface | Signal / Protocol | Conditioning & Protection | Compatibility |
| :--- | :--- | :--- | :--- | :--- |
| **`J2` / `J3`** | `XT30PW-M` (2-pin) | Power $V_{\text{IN}}$ / Motor Out | `SMBJ15CA` TVS, N-MOSFETs, polarity/PWM LEDs | 2S–3S LiPo, brushed DC motor |
| **`J11`** | `JST-GH` (3-pin) | Half-Duplex RS-485 ($\le 500\text{ kbps}$) | `THVD1400D`, $120\ \Omega$ termination, `SM712` TVS | Multi-axis bus, telemetry |
| **`J7` / `J8`** | `JST-GH` (2-pin) | $2\times$ Isolated PWM / GPIO ($3.3\text{–}5\text{ V}$) | `TLP2310` optocouplers, `SMF6.0A` reverse clamp | PWM/RC setpoint, redundant input |
| **`J12`** | `JST-GH` (6-pin) | Hybrid `SPI2` or `I2C1` Bus | `ESD224DQA` TVS ($0.5\text{ pF}$), $+3.3\text{ V}$ ($22\ \Omega$), `DNP` pull-ups | **AS5047P**, **AS5048A**, **MA730**, **AS5600** |
| **`J10`** | `JST-GH` (4-pin) | Quadrature Encoder A/B ($3.3\text{–}5\text{ V}$) | `USBLC6-2SC6`, $+3.3\text{ V}$ ($22\ \Omega$), RC ($160\text{ kHz}$) + Schmitt | Incremental Hall-effect encoders |
| **`J4` / `J5`** | `JST-GH` (2-pin) | $2\times$ Limit Switches (dry contact) | $2.2\text{ k}\Omega$ pull-up, RC ($160\text{ Hz}$) + Schmitt | Min / Max mechanical endstops |
| **`J9`** | `JST-GH` (3-pin) | Board Enable (`SHDN_EXT`) | Direct control of `LTC4365` `SHDN` pin | Power / sleep switch |
| **`J6`** | `JST-GH` (2-pin) | Power Fault Output (`NFAULT_EXT`) | Active-low open-drain (`LTC4365`) | Host system supervision |
| **`J1`** | `TC2070-NL` (14-pin) | SWD + NRST + VCP UART (`USART3`) | No-legs footprint (STDC14 compatible) | ST-LINK/V3 (flashing & debug) |

For absolute position sensors on **`J12`**, the port operates by default in **high-speed SPI mode** (recommended in noisy motor environments for **AS5047P** / **MA730**). It can be switched to **I²C mode** for an **AS5600** by populating the two $4.7\text{ k}\Omega$ pull-up resistors on `Connector.kicad_sch`.

---

## 4. MCU Pinout (`STM32G070CBT6` — LQFP48)

Pin assignments are grouped by physical package edge to keep all H-bridge control signals along the right side of the LQFP48, isolate analog sensing channels (`ADC_VM` and `IPROPI`), and eliminate trace crossovers to peripheral connectors.

| Functional Block | Net Name | Pin | Peripheral | Role |
| :--- | :--- | :---: | :--- | :--- |
| **Motor Driver** | `PH/IN2_MCU` | `PA8` (28) | `TIM1_CH1` | Direction (`PH`) or secondary PWM |
| | `EN/IN1_MCU` | `PA9` (29) | `TIM1_CH2` | High-frequency motor PWM ($32\text{ kHz}$) |
| | `NSLEEP_MCU` | `PC6` (30) | `GPIO_Output` | H-bridge enable / sleep control |
| | `PMODE_MCU` | `PC7` (31) | `GPIO_Output` | `PH/EN` (`0`) or `PWM` (`1`) mode select |
| | `NFAULT_MCU` | `PA10` (32) | `GPIO_EXTI10` | Driver fault interrupt (falling edge) |
| | `IPROPI_MCU` | `PB10` (22) | `ADC1_IN11` | Motor current sense ($1.13\text{ V/A}$, RC filtered) |
| **Supervision** | `ADC_VM` | `PB2` (21) | `ADC1_IN10` | Protected input bus voltage $V_M$ sense |
| **RS-485** | `RS485_DE` / `TX` / `RX` | `PA1`, `PA2`, `PA3` | `USART2` (+ DMA) | Differential bus with hardware DE control |
| **Encoder** | `TIM_ENC_A` / `B` | `PA6` (17), `PA7` (18) | `TIM3_CH1/CH2` | Hardware X4 quadrature decoder |
| **Optocouplers** | `OPTO1_MCU` / `OPTO2_MCU` | `PB14` (26), `PB15` (27) | `TIM15_CH1/CH2` | PWM Input Capture / redundant setpoint |
| **Limit Switches** | `TRIG1_MCU` / `TRIG2_MCU` | `PD2` (40), `PD3` (41) | `GPIO_EXTI2/EXTI3` | Endstop interrupts (rising edge) |
| **Sensor Bus** | `MISO`/`MOSI`/`SCK`/`NSS` | `PB6`..`PB9` (45–48) | `SPI2` (or `I2C1`) | Absolute magnetic encoder interface |
| **HMI** | `BLUE` / `RED` / `GREEN` | `PC13`, `PC14`, `PC15` | `GPIO_Output` | 8-state RGB status LED (common anode, active-low) |
| | `BUZZER_MCU` | `PA4` (15) | `TIM14_CH1` | $4\text{ kHz}$ piezo buzzer PWM drive |
| **Debug** | `SWDIO` / `SWCLK` | `PA13` (35), `PA14` (36) | `SYS_SWD` | Programming & debugging |
| | `VPC_TX` / `VPC_RX` | `PA5` (16), `PB0` (19) | `USART3_TX/RX` | Virtual COM port diagnostic console |

---

## 5. KiCad Architecture & Repository Structure

The hardware design is structured hierarchically in **KiCad 10.0** with a root sheet and three functional sub-sheets, paired with the **STM32CubeMX** configuration file (`KinetiCoreDC.ioc`):

```text
KinetiCoreDC/
├── KinetiCoreDC.kicad_pro       # KiCad 10.0 project file
├── KinetiCoreDC.kicad_sch       # Sheet 1/4: STM32G070CBT6 MCU & hierarchy
├── Interface.kicad_sch          # Sheet 2/4: RS-485, optocouplers, encoder, endstops
├── Power.kicad_sch              # Sheet 3/4: LTC4365, 3.3V LDO, DRV8874, RGB, buzzer
├── Connector.kicad_sch          # Sheet 4/4: XT30, JST-GH, Tag-Connect, ESD224DQA
├── KinetiCoreDC.kicad_pcb       # PCB layout
├── KinetiCoreDC.ioc             # STM32CubeMX hardware configuration
├── Datasheet/                   # Key component datasheets
├── lib/                         # Local symbols, footprints, and 3D models
├── README.md                    # French documentation
├── README.en.md                 # English documentation
└── LICENSE                      # CERN-OHL-S-2.0 license
