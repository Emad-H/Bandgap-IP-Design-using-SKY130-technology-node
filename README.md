# Bandgap IP Design using Sky130 technology node

![workshop-flyer](Workshop-Flyer.png)

A 2 day cloud based virtual training workshop conducted by VSD-IAT from 20<sup>th</sup> to 21<sup>st</sup> November, 2021. The link to the workshop webpage can be found [here](https://www.vlsisystemdesign.com/bandgap-ip-design-using-sky130-technology-node/). Below is a brief, day-wise documentation about the topics covered in the course, along with my implementations of the lab sessions.


## Table of Contents

  * [Day 1 - Bandgap Design Theory](#day-1---bandgap-design-theory)
    + [Introduction to Bandgap Voltage Reference](#introduction-to-bandgap-voltage-reference)
    + [CTAT Voltage Generation Circuit](#ctat-voltage-generation-circuit)
  * [Day 2](#)
    + [](#)
  * [Acknowledgements](#acknowledgements)

## Day 1 - Bandgap Design Theory

Integrated circuits or System on Chip (SoC) generally contain many analog and digital subsystems requiring various regulated supply voltages. These supply voltages are provided by linear voltage regulators such as LDOs. The LDOs, as well as some subcircuits like ADCs or DACs, need a reference voltage input that must be independent to variations in process, voltage and temperature. Typical power supplies can create noisy/rippled outputs, or may have their voltage drop over time like batteries. A Process-Voltage-Temperature Independent Biasing Circuit is used to provide this reference voltage, with typical values as follows:
 + Typical Temperature Coefficient = 10 to 50 ppm/°C (~ 10 to 50 uV/°C for <i>V<sub>REF</sub></i> = 1V)
 + Typical Power Supply Rejection Ratio = 40 to 60 dB (10-1 mV/V)

Similarly, this PVT independent biasing circuit can be used to provide reference currents as well to the various subsystems.

Some methods to generate reference voltages are as follows:

1. **Voltage Divider Circuit** - Good temperature coefficient, but undesirable power supply sensitivity
2. **Forward-Biased PN Junction** - Worse temperature coefficient (~3142 ppm/°C), but better power supply sensitivity than a voltage divider
3. **Base-Emitter Voltage Referenced Circuit** - Good supply rejection due to the use of current mirror circuit, but undesirable temperature coefficient (~2333 ppm/°C)
4. **Bandgap Voltage Reference Circuit** - Great temperature coefficient (~10-50 ppm/°C) and power supply sensitivity

### Introduction to Bandgap Voltage Reference

A bandgap voltage reference (BGR) circuit works on the principle that equally adding 2 voltages, one with a negative temperature coefficient and one with a positive temperature coefficient, should give a voltage reference that is independent to variations in temperature. In practice however, it is generally in the range of 10 to 50 ppm/°C. This is shown below.

![bg-cicruit-principle](Day1/1-0.png) ![bg-cicruit-principle2](Day1/1-1.png)

Some applications of a BGR are listed below:
- Low Dropout Regulator (LDO)
- DC-to-DC Buck Converter
- Analog-to-Digital Converter (ADC)
- Digital-to-Analog Converter (DAC)

**Types of BGR:**
- Based on Architecture
  1. Using Self-Biased Current Mirror
  2. Using Operational-Amplifier
- Based on Application
  1. Low-Voltage BGR
  2. Low-Power BGR
  3. High PSRR and Low-Noise BGR
  4. Curvature Compensated BGR

**Self-Biased Current Mirror based BGR**

This is the topology that will be designed during the course of this workshop. It has the following advantages:
- Simplest topology
- Easy to design
- Always stable

Some limitations for this design are as follows:
- Low power supply rejection ratio (PSRR)
- Cascode design needed to reduce PSRR
- Voltage head-room issue
- Needs a start-up circuit

The following components are required to develop a BGR:
- CTAT voltage generation circuit
- PTAT voltage generation circuit
- Self-biased current mirror circuit
- Reference branch circuit
- Start-up circuit

### CTAT Voltage Generation Circuit

























## Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123)
- [VSD-IAT](https://vsdiat.com/)
