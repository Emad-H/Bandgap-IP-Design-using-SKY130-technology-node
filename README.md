# Bandgap IP Design using Sky130 technology node

![workshop-flyer](Workshop-Flyer.png)

A 2 day cloud based virtual training workshop conducted by VSD-IAT from 20<sup>th</sup> to 21<sup>st</sup> November, 2021. The link to the workshop webpage can be found [here](https://www.vlsisystemdesign.com/bandgap-ip-design-using-sky130-technology-node/). Below is a brief, day-wise documentation about the topics covered in the course, along with my implementations of the lab sessions.


## Table of Contents

  * [Day 1 - Bandgap Design Theory](#day-1---bandgap-design-theory)
    + [Introduction to Bandgap Voltage Reference](#introduction-to-bandgap-voltage-reference)
    + [CTAT Voltage Generation Circuit](#ctat-voltage-generation-circuit)
    + [PTAT Voltage Generation Circuit](#ptat-voltage-generation-circuit)
    + [Self-Biased Current Mirror Circuit](#self-biased-current-mirror-circuit)
    + [Start-Up Circuit](#start-up-circuit)
    + [Complete BGR Circuit](#complete-bgr-circuit)
  * [Day 2 - Bandgap Design using SKY130](#day-2---bandgap-design-using-sky130)
    + [Tools and PDK Setup](#tools-and-pdk-setup)
    + [Design Specifications, Device Data and Design Steps](#design-specifications-device-data-and-design-steps)
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

![ctat-gen](Day1/1-2.png)

Diodes have a negative temperature coefficient, but we make use of a BJT as a diode by shorting the base and collector (for PNP diode). This is because diodes have their P substrate connected to GND in a SoC, which is an invalid configuration for a CTAT circuit. This is shown below.

![diode-struct](Day1/1-3.png)

A regular BJT used in diode configuration as shown in (b) can cause a parasitic BJT to be formed, making it unusable for the CTAT circuit. Hence, we make use of the structure in figure (c) as a BJT in diode configuration to generate the CTAT voltage.

We can now design the CTAT circuit as shown below, and further analyse its temperature coefficient as shown.

![ctat-analysis](Day1/1-4.png)

The figure below shows how variations in the CTAT circuit design affect its temperature coefficient.

![ctat-gen](Day1/1-5.png)

Here, figure (b) shows that adding more BJT units further increases the negative slope for V<sub>CTAT</sub> vs. temperature for the circuit. We can also alter the slope by changing the current through the BJT, as shown in figure (c). As we decrease the current through the BJT, the slope increases negatively.

### PTAT Voltage Generation Circuit

A PTAT voltage generation circuit uses multiple BJTs in diode configuration to create a voltage with positive temperaure coefficient. Its working principle is shown in the figure below.

![ptat-working](Day1/1-6.png)

Here, Q1 is a single BJT, while Q2 has multiple BJTs in parallel in the ratio 1:N to Q1. We can further analyse the voltages V and V<sub>1</sub> as follows.

![ptat-slope](Day1/1-7.png)

We can see above that V<sub>1</sub> has a more negative slope than V, as it has N BJTs connected in parallel. This causes the current to be divided, leading to lower current through each BJT. Now, if we take the difference of the voltages V and V<sub>1</sub>, we get a voltage with a positive slope versus temperature.

**Design of Resistor R1**

We must also design the value of the resistor R1 according to the power consumption specifications and silicon area budget. An increase in resistance value will cause increased silicon area, but lower current. The value of the resistance also depends on the number of BJTs used. The equation to calculate R1 is given below.

R1 = V<sub>t</sub> &sdot; ln (N)/I

<!---<img src="https://render.githubusercontent.com/render/math?math=R1 = V_{t} \frac{ln (N)}{I}"> --->

### Self-Biased Current Mirror Circuit

In a PTAT generation circuit, we make use of a self-biased current mirror to ensure that the node voltages and currents are the same. A current mirror allows us to establish a DC voltage or current that is independent of supply and process, with a well-defined behaviour with temperature. A simple current mirror is shown below.

![curr-mirror](Day1/1-8.png)

The issue with a simple current mirror is that the output current is very sensitive to *V<sub>DD</sub>*. We can also use a resistor R<sub>1</sub> like shown in the figure. This, however, will allow variations in the current with changes in temperature, as resistors are temperature dependent. For a less sensitive solution, we must use a circuit that biases itself. This means *I<sub>REF</sub>* must be somehow derived from *I<sub>OUT</sub>*.

![selfbias-curr-mirror](Day1/1-9.png)

The figure above shows a self-biased current mirror. The idea of self-biasing is that if *I<sub>OUT</sub>* can be independent of *V<sub>DD</sub>*, then *I<sub>REF</sub>* can be a replica of *I<sub>OUT</sub>*. Here, MP<sub>1</sub> and MP<sub>2</sub> copy *I<sub>OUT</sub>* and define *I<sub>REF</sub>*, meaning *I<sub>REF</sub>* is bootstrapped to *I<sub>OUT</sub>*. Since each diode-connected device feeds from a current source, *I<sub>OUT</sub>* and *I<sub>REF</sub>* are ultimately independent of *V<sub>DD</sub>*.

In this circuit however, the currents are given by the equation I<sub>OUT</sub> = K &sdot; I<sub>REF</sub> and *I<sub>OUT</sub>* cannot be uniquely defined. We can fix this using the circuit below.

![selfbias-fix](Day1/1-10.png)

Here, R<sub>S</sub> allows us to choose a value of *I<sub>OUT</sub>* as desired. Also, since the loop-gain is now less than 1, the circuit is always stable. However, this creates a start-up issue in the circuit due to the existence of a new degenerate bias point. This means when a supply voltage is given, the circuit may still have zero current flowing, and so  we must use a start-up circuit to fix this. Finally, the entire circuit is shown below (without start-up circuit).

![selfbias-full](Day1/1-11.png)

### Reference Voltage Branch Circuit

![refvolt-ckt](Day1/1-12.png)

In the figure above, we see both the CTAT and PTAT generation circuits used to generate the reference voltage *V<sub>REF</sub>*. From the circuit, we can observe the following:
- Current I<sub>3</sub> is the same as I<sub>1</sub> & I<sub>2</sub>
- Voltage across Q<sub>3</sub> is CTAT type
- Voltage across R<sub>2</sub> is PTAT type
- V<sub>REF</sub> is the addition of CTAT and PTAT voltages
- R<sub>2</sub> = α &sdot; R<sub>1</sub>

**Design of Resistance R<sub>2</sub>**

In order to get a temperature coefficient of 0 for V<sub>REF</sub>, we must adjust the value of R<sub>2</sub> such that R<sub>2</sub> = α &sdot; R<sub>1</sub>. We can find the value of α as follows:

![r2-aplha](Day1/1-13.png)

where,

![r2-where](Day1/1-14.png)

This way, we can get an output reference voltage V<sub>REF</sub> with 0 temperature coefficient at nominal temperatures and slight variations at higher temperatures.

### Start-Up Circuit

If the channel length modulation is negligible, the current hardly depends on the supply voltage. The main issue with supply independent biasing is the existence of degenerate bias points. There are two stable operating points:
- I<sub>in</sub> = I<sub>out</sub> = 0A (undesired operating point)
- Desired operating points

We must keep the circuit out of the undesired operating point when the supply is turned on, and must not interfere with the circuit once it reaches the desired operating point. We can do this by adding a start-up circuit as shown below.

![start-up-ckt](Day1/1-15.png)

Initially, the current in every branch remains zero. Because of this, the voltage at net2 follows the voltage V<sub>DDD</sub>, while voltage at net1 will be nearly zero. In order to cause a current flow in net1, we use the transistor MP<sub>5</sub>. To create a current flow through MP<sub>5</sub>, there must be a voltage difference greater than V<sub>t</sub> accross its source and gate terminals. This is done by the transistor MN<sub>3</sub>.

As current flows through net6, transisitor MN<sub>3</sub> creates a voltage difference of greater than V<sub>t</sub> accross transistor MP<sub>5</sub>, allowing current through net1. This current gets mirrored to net 2, causing the circuit to move into a desired stable point. As current stably flows through net 2, the voltage at net2 goes down, causing transistor MP<sub>5</sub> to go into reverse bias. This turns off transistor MP<sub>5</sub>, allowing the start-up circuit to isolate itself from the reference voltage circuit (however, some minimal current will always flow through net6).

### Complete BGR Circuit

Finally, the complete bandgap reference circuit is shown below.

![complete-bgr](Day1/1-16.png)

<!-- > Note: MP<sub>1</sub> and MP<sub>2</sub> must be kept at saturation. MP<sub>3</sub> can be identical to both MP<sub>1</sub> and MP<sub>2</sub>. Q<sub>3</sub> can also be the same as Q<sub>1</sub>. -->

## Day 2 - Bandgap Design using SKY130

A typical Analog Design Flow is shown below. Here, DRC stands for design rule checks and PEX stands for parasitics extraction.

![design-flow](Day2/2-0.png)

### Tools and PDK Setup

The tools we shall use are NgSpice, Magic and Netgen; along with the Google SkyWater Open Source PDKs.

**Setting up SkyWater PDK Libraries and Tech Files**

We can download the PDK libraries for primitives, available [here](https://github.com/google/skywater-pdk-libs-sky130_fd_pr), by running the command `git clone https://github.com/google/skywater-pdk-libs-sky130_fd_pr.git` as shown below.

![git-clone](Day2/2-1.png)

This command should add device models and related files for all primitive devices in the SkyWater PDK libraries onto your home directory.

Next, we need to download the Magic tech file for layout design and Netgen tech file for LVS. These can be found [here](https://github.com/silicon-vlsi-org/eda-technology) and we can download them with the command `git clone https://github.com/silicon-vlsi-org/eda-technology.git` as follows.

![git-clone2](Day2/2-2.png)

**Running Magic and Netgen**

We must open Magic while specifiying the tech files provided in the PDK library. This can be done using the following command.

![mag-cmd](Day2/2-3.png)

Similarly, we can call the Netgen console interface using the command `netgen` in the terminal, and run LVS using the following command, while specifying the tech file.

![netgen-cmd](Day2/2-4.png)

### Design Specifications, Device Data and Design Steps

Let us design a bandgap reference circuit with the following design specifications:
- Supply voltage = 1.8V
- Temperature: -40&deg;C to 125&deg;C
- Power Consumption < 60&mu;W
- Off current < 2&mu;A
- Start-up time < 2&mu;s
- Temperature coefficient Of V<sub>REF</sub> < 50 ppm

For this design, we shall be using mainly 3 types of devices: MOSFETs, BJTs and Resistors. Let us look at the datasheets for each of the devices.

**1. MOSFET**

|Parameter|NFET|PFET|
|:---|:---:|:---:|
|Type|LVT	|LVT|
|Voltage	|1.8 V|	1.8 V|
|Threshold Voltage	|~ 0.4 V	 |~ -0.6 V |
|Model	Name|sky130_fd_pr__nfet_01v8_lvt	|sky130_fd_pr__pfet_01v8_lvt|

**2. BJT**

|Parameter	|PNP|
|:---|:---:|
|Current Rating|	1 &mu;A/&mu;m<sup>2</sup> to 10 &mu;A/&mu;m<sup>2</sup>|
|Beta	|~ 12|
|Emitter Area	|11.56 &mu;m<sup>2</sup>|
|Model Name|	sky130_fd_pr__pnp_05v5_W3p40L3p40|

**3. Resistor**

|Parameter|	RPOLYH|
|:---|:---:|
|Sheet Resistance|	~ 350 &#x2126;|
|Temperature Coefficient	|2.5 &#x2126;/&deg;C|
|Bin Width	|0.35 &mu;, 0.69 &mu;, 1.41 &mu;, 5.37 &mu;|
|Model	Name|sky130_fd_pr__res_high_po|

Next, we shall adhere to the following Design Steps:

**Step 1. Calculation of current**

**Step 2. Choosing the number of BJTs in parallel for branch 2**'

**Step 3. Calculation of resistance R<sub>1</sub>**

**Step 4. Calculation of resistance R<sub>2</sub>**

**Step 5. PMOS design for self-biased current mirror**

**Step 6. NMOS design for self-biased current mirror**

Finally, we get the following completed BGR circuit.

![complete-bgr-design](Day2/2-5.png)

### PTAT Design and Pre-Layout Simulation

![ptat-schem](Day2/2-6.png)

Given above is a PTAT voltage generation circuit chematic using a VCVS. Let us write a spice netlist for the same, as shown below.

![ptat-sp-net](Day2/2-7.png)

Next, we can simulate this netlist in NgSpice using the command `ngspice ptat_voltage_gen.sp`. We should now see the following in our terminal.

![ptat-sp-cmd](Day2/2-8.png)

Now, let us plot the variation of different net voltages with temperature. We can do this using the `plot v(qp1) v(ra1) v(qp2) v(net2)` command.

![ptat-sim-plot](Day2/2-9.png)

As we can see, the voltage at qp1 and qp2 are both CTAT in nature, but have different slopes. The difference of the two voltages is PTAT in nature, and can be seen using the command `plot v(ra1)-v(qp2)`. We can observe this clearly using the plot below.

![ptat-sim-plot2](Day2/2-10.png)

If we find the slope of the PTAT curve, we get the following.

![ptat-sim-slope](Day2/2-11.png)

Finally, we can plot the currents in both branches to confirm that they have equal current flowing through them.

![ptat-sim-current-plot](Day2/2-12.png)

### CTAT Design and Pre-Layout Simulation

**CTAT with Single BJT**

Similarly, we can write a spice netlist for a simple CTAT circuit using a BJT in diode configuration, as shown below.

![ctat-sp-net](Day2/2-13.png)

Next, we can simulate the spice netlist using NgSpice with command `ngspice ctat_voltage_gen.sp`. We should see the simulation taking place in the console as shown.

![ctat-sim-cmd](Day2/2-14.png)

We can display the plot for voltage at node qp1 by running the command `plot v(qp1)` in the terminal. We should see the following.

![ctat-sim-plot](Day2/2-15.png)

If we inspect the slope from our plot, we can find that the temperature coefficient for a single BJT is as follows.

![ctat-sim-slope](Day2/2-16.png)

**CTAT with Multiple BJTs**

We can write a spice netlist for a CTAT circuit using 8 BJTS in diode configuration, as shown below.

![ctat2-sp-net](Day2/2-17.png)

Let us simulate this netlist in NgSpice and plot the voltage at node qp1 as shown below.

![ctat2-sim-plot](Day2/2-18.png)

Finally, if we calculate the slope, we can find that the slope increases negatively as we increase the number of BJTs.

![ctat2-sim-slope](Day2/2-19.png)

**CTAT with Different Current Values**

Now, let us see what happens to the temperature coefficient as we vary the current through the BJT. We can write a spice netlist for this as follows.

![ctat3-sp-net](Day2/2-20.png)

Now, we can simulate this in NgSpice and plot the voltage at qp1.

![ctat3-sim-plot](Day2/2-21.png)

Frome the above plot, we can see the effect of varying the current through the BJT on the temperature coefficient of CTAT voltage. As the current dereases, the slope increases negatively.

### Ideal BGR Design and Pre-Layout Simulation















## Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123)
- [VSD-IAT](https://vsdiat.com/)
