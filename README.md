# Motor-driver-with-ethercat


A custom three-phase motor controller based on the **STM32G474**, **DRV8353** gate driver, and **LAN9252 EtherCAT Slave Controller**.

The hardware integrates a three-phase MOSFET inverter, current sensing, magnetic encoder feedback, temperature monitoring, CAN communication, USB, and EtherCAT connectivity.

## Hardware Features

- STM32G474VET3 microcontroller
- DRV8353FSRTAR three-phase gate driver
- Six ISC008N06LM6 power MOSFETs
- Three-phase current sensing using 1 mΩ shunt resistors
- Back-EMF sensing for all three motor phases
- LAN9252 EtherCAT slave controller
- Dual Ethernet ports
- AS5047D magnetic rotary encoder
- Three NTC temperature-sensing channels
- MCP2562FD CAN transceiver
- USB Type-C interface
- Onboard 5 V buck converter
- 3.3 V LDO power supply
- DC-link capacitor bank
- SWD, UART and BOOT interfaces

## 1. System Architecture

| Subsystem | Main Component | Function |
|---|---|---|
| Main Controller | STM32G474VET3 | Motor control and communication |
| Gate Driver | DRV8353FSRTAR | MOSFET gate driving |
| Power Stage | ISC008N06LM6 | Three-phase inverter |
| EtherCAT | LAN9252V/ML | Industrial Ethernet communication |
| Encoder | AS5047D | Rotor-position feedback |
| CAN | MCP2562FD | CAN communication |
| Buck Converter | LMR51635 | 5 V power generation |
| LDO | TPS7A7001 | 3.3 V power generation |
| Temperature Sensors | NCP18XH103F03RB | Temperature measurement |

## 2. Power Supply

### 2.1 Main DC Input

The motor supply enters through an XT90H connector.

Components:
- XT90H-F.G.Y power connector
- SMCJ33A TVS diode
- SF-2923HC50C-2 input switching component
- Battery voltage sensing circuit
- DC-link capacitor bank

### 2.2 Buck Converter

The onboard buck converter uses the LMR51635XDDCR regulator.

Main components:
- LMR51635XDDCR
- 8.2 µH inductor
- 4.7 µF input capacitors
- 100 nF bypass capacitor
- Two 47 µF output capacitors
- Feedback resistors

The output supply is labelled LM_5V.

### 2.3 3.3 V LDO

The TPS7A7001DDAR LDO generates the 3.3 V logic supply.

It supplies the low-voltage electronics and control interfaces.

### 2.4 DC-Link Capacitors

The DC-link capacitor bank consists of:

| Reference | Value | Quantity |
|---|---:|---:|
| C67–C69 | 680 µF | 3 |
| C70–C72 | 10 µF | 3 |
| C73–C75 | 2.2 µF | 3 |
| C76–C78 | 100 nF | 3 |



## 3. Three-Phase MOSFET Inverter

### 3.1 Power Stage

The inverter uses six ISC008N06LM6 N-channel MOSFETs arranged in three half-bridges.

| Motor Phase | High-Side MOSFET | Low-Side MOSFET |
|---|---|---|
| Phase A | Q1 | QN1 |
| Phase B | Q2 | QN2 |
| Phase C | Q3 | QN3 |

Motor phase outputs:
- PHASE-A
- PHASE-B
- PHASE-C

### 3.2 Gate Driver

The DRV8353FSRTAR controls the six MOSFET gates.

| Signal | Function |
|---|---|
| GHA | Phase A high-side gate |
| GLA | Phase A low-side gate |
| GHB | Phase B high-side gate |
| GLB | Phase B low-side gate |
| GHC | Phase C high-side gate |
| GLC | Phase C low-side gate |

The STM32 communicates with the DRV8353 through SPI.

The driver also provides fault monitoring and current-sense amplifier outputs.

### 3.3 Current Sensing

Three 1 mΩ shunt resistors are used for phase-current measurement.

| Reference | Phase |
|---|---|
| R25 | Phase A |
| R22 | Phase B |
| R23 | Phase C |

The DRV8353 provides three current-sense amplifier outputs:

- SOA
- SOB
- SOC

These signals are used for current-feedback measurement.

### 3.4 Back-EMF Sensing

Three sensing channels are provided:

- Bemf-A
- Bemf-B
- Bemf-C

Each channel contains a resistor network, diode protection and filtering capacitor.

These circuits provide conditioned phase-voltage signals for measurement.

## 4. STM32G474 Microcontroller

The main controller is the STM32G474VET3.

It interfaces with:
- DRV8353 gate driver
- LAN9252 EtherCAT controller
- AS5047D magnetic encoder
- CAN transceiver
- USB interface
- Battery voltage sensing
- Back-EMF sensing
- Temperature sensing

### 4.1 Gate Driver Interface

| Signal | Function |
|---|---|
| DRV_CS | SPI chip select |
| DRV_SCLK | SPI clock |
| DRV_SDI | SPI data input |
| DRV_SDO | SPI data output |
| ENABLE | Gate-driver enable |
| drv_nFAULT | Fault indication |

The six PWM control signals are INHA, INLA, INHB, INLB, INHC and INLC.

### 4.2 Clock and Debugging

The MCU section includes:
- External 8 MHz crystal
- SWD debugging interface
- BOOT selection header
- Reset circuit
- UART communication header

### 4.3 MCU Power Filtering

The MCU power section includes local decoupling capacitors.

The analog power section uses ferrite beads and capacitors to filter the VDDA and VREF+ supply domains.

## 5. EtherCAT Communication

The EtherCAT interface is based on the LAN9252V/ML EtherCAT Slave Controller.

### 5.1 Main Components

- LAN9252V/ML
- 25 MHz DSC1001 oscillator
- 24FC512 EEPROM
- Two HR913550A Ethernet connector modules
- Ethernet termination and filtering components

### 5.2 STM32 to LAN9252 Interface

| Signal | Function |
|---|---|
| CAT_CS | Chip select |
| CAT_SCK | Serial clock |
| SIO0 | Serial data |
| SIO1 | Serial data |
| SIO2 | Serial data |
| SIO3 | Serial data |
| IRQ | Interrupt request |
| RST | Reset |
| SYNC0/LATCH0 | Synchronization |
| SYNC1/LATCH1 | Synchronization |

The host-interface operating mode is determined by the LAN9252 configuration and firmware.

### 5.3 Ethernet Ports

Two Ethernet ports are provided:

- PORT-A
- PORT-B

The dual-port arrangement supports EtherCAT line-topology connections.

### 5.4 EEPROM

The 24FC512 EEPROM connects to the LAN9252 through EESDA and EESCL.

It is intended to store EtherCAT slave configuration information.

## 6. Magnetic Encoder

The motor-position feedback circuit uses the AS5047D magnetic rotary encoder.

### 6.1 Encoder Interface

| Signal | Function |
|---|---|
| ENCO_CS | SPI chip select |
| ENCO_CLK | SPI clock |
| ENCO_MISO | Encoder data output |
| ENCO_MOSI | Encoder data input |

The encoder is supplied through the 3.3 V supply arrangement shown in the schematic.

A 100 nF capacitor provides local decoupling.

The encoder provides rotor-position feedback for closed-loop motor control.

## 7. Temperature Monitoring

Three NTC thermistor channels are provided.

**Thermistor:** NCP18XH103F03RB

Each channel contains:
- 10 kΩ pull-up resistor
- NTC thermistor connected to ground
- Voltage-divider measurement point

| Channel | Pull-up | Thermistor |
|---|---|---|
| T1 | R55 | R54 |
| T2 | R57 | R56 |
| T3 | R59 | R58 |

The voltage-divider outputs can be measured using MCU ADC channels.

## 8. CAN Communication

The CAN interface uses the MCP2562FD-E/SN transceiver.

### Signals

- CAN_TX
- CAN_RX
- CANH
- CANL
- STBY

The schematic includes a 120 Ω termination resistor.

Termination population depends on the board's position in the CAN network.

## 9. USB and Programming Interfaces

The board includes a USB Type-C connector.

USB data signals:
- DP
- DN

The USBLC6-2SC6 provides USB data-line ESD protection.

Additional interfaces:
- SWD programming and debugging
- UART communication
- BOOT selection
- RESET circuit

## 10. Motor Control Signal Flow

The intended hardware signal flow is:

1. DC input supplies the inverter and onboard power regulators.
2. STM32G474 generates the motor-control PWM signals.
3. DRV8353 drives the six MOSFET gates.
4. The three-phase inverter supplies the motor phases.
5. Current-sense shunts provide current-feedback signals.
6. AS5047D provides rotor-position feedback.
7. LAN9252 provides EtherCAT communication with an external master.
8. NTC channels provide temperature-monitoring inputs.


## Disclaimer

This is a custom motor-controller hardware design. Electrical characteristics, maximum current, operating voltage, protection behaviour and firmware functionality must be verified through component datasheets, PCB review and hardware testing before deployment.
