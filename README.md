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
    + [PTAT Design and Pre-Layout Simulation](#ptat-design-and-pre-layout-simulation)
    + [CTAT Design and Pre-Layout Simulation](#ctat-design-and-pre-layout-simulation)
    + [BGR using Ideal Op-Amp (VCVS)](#bgr-using-ideal-op-amp-vcvs)
    + [Complete BGR Circuit using SBCM](#complete-bgr-circuit-using-sbcm)
    + [Detailed Plots for Start-up Simulation](#detailed-plots-for-start-up-simulation)
    + [Layout Design](#layout-design)
    + [Top Level Layout Extraction, LVS and Post-Layout Simulation](#top-level-layout-extraction-lvs-and-post-layout-simulation)
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

The tools we shall use are NgSpice, Magic and Netgen; along with the Google SkyWater Open Source PDKs. All prerequisite layout and spice model files can be found [here](https://github.com/vsdip/vsdopen2021_bgr).

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

### BGR using Ideal Op-Amp (VCVS)

![bgr-vcvs-schem](Day2/2-22.png)

Above, we have used an ideal operational amplifier (VCVS) to design a BGR circuit. We have used the MOS dimensions for tranisitors MP<sub>1</sub>, MP<sub>2</sub> and MP<sub>3</sub> to allow the same current through all branches. In order to make the CTAT and PTAT voltages equal, the gains must be set accordingly. We do this by adjusting the values of R<sub>2</sub> and R<sub>1</sub>. Here, we have picked an &aplha; value of approximately 9. We can now write the spice netlist for this as follows.

![bgr-net1](Day2/2-23.png)
![bgr-net2](Day2/2-24.png)

Here, we have used 2 series and 2 parallel resistors of 2 K&#x2126; value to create the 5 K&#x2126; resistance for R<sub>1</sub>, and 22 series and 2 parallel resistors of 2 K&#x2126; value to create the 45 K&#x2126; resistance for R<sub>2</sub>.

Next, let us simulate this netlist in NgSpice as follows and plot the voltage V<sub>REF</sub>.

![bgr-plot-vref](Day2/2-25.png)

We can observe an umbrella shaped curve, as expected. As temperature changes, the voltage does not vary much. If we calculate the ppm, we get approximately 30 ppm, which adheres to our requirement.

We can also plot the curve for voltage at qp3 (CTAT) and find its slope.

![bgr-plot-vqp3](Day2/2-26.png)
![bgr-plot-vqp3-slp](Day2/2-27.png)

Now, if we plot the voltage from the PTAT generation circuit, we find that the slope is almost the same, but opposite polarity.

![bgr-plot-vqp3ref](Day2/2-28.png)
![bgr-plot-vqp3ref-slp](Day2/2-29.png)

Further, we can plot all the voltages, and we get following.

![bgr-plot-all1](Day2/2-30.png)

![bgr-plot-all2](Day2/2-31.png)

Finally, we can check the slope for temperature coefficient of voltage across the resistor R<sub>1</sub> and confirm that it is indeed approximately equal to -1/9<sup>th</sup> of the temperature coefficient of the CTAT voltage generated across R<sub>2</sub>.

![bgr-plot-r1](Day2/2-32.png)

![bgr-plot-r1-slp](Day2/2-33.png)

### Complete BGR Circuit using SBCM

![bgr-comp-schem](Day2/2-34.png)

Above, we have a complete BGR circuit that makes use of a self-biased current mirror and start-up circuit.

We shall perform multiple types of analysis on this circuit.

**1. DC Analyis vs. Temperature at tt corner**

```verilog
**** bandgap reference circuit using self-biase current mirror by emad h. *****

.lib "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
.dc	temp	-40	125	5

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
```

We can now simulate and plot the curves for V<sub>REF</sub>, various node voltages and branch currents.

![bgr-comp-dc1](Day2/2-35.png)
![bgr-comp-dc2](Day2/2-36.png)
![bgr-comp-dc3](Day2/2-37.png)

If we observe from the plots, we get a temperature coefficient of approximately 22 ppm/&deg;C.

**2. DC Analyis vs. Temperature at ss corner**

```verilog
**** bandgap reference circuit using self-biase current mirror at ss corner by emad h. ****

.lib "/home/santunu/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice ss"
.include "/home/santunu/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
.dc	temp	-40	125	5

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
```

We can now simulate and plot the curves for V<sub>REF</sub>, various node voltages and branch currents.

![bgr-comp-ss1](Day2/2-38.png)
![bgr-comp-ss2](Day2/2-39.png)
![bgr-comp-ss3](Day2/2-40.png)

If we observe from the plots, we get a temperature coefficient of approximately 43 ppm/&deg;C.

**3. DC Analyis vs. Temperature at ff corner**

```verilog
**** bandgap reference circuit using self-biase current mirror at ff corner by emad h. *****

.lib "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice ff"
.include "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	dc 	2
.dc	temp	-40	125	5

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
```

We can now simulate and plot the curves for V<sub>REF</sub>, various node voltages and branch currents.

![bgr-comp-ff1](Day2/2-41.png)
![bgr-comp-ff2](Day2/2-42.png)
![bgr-comp-ff3](Day2/2-43.png)

If we observe from the plot for V<sub>REF</sub>, we find that it is internally compensated, and we get a temperature coefficient of approximately 10.1 ppm/&deg;C. This make ff the best corner.

**4. Transient Analysis at tt corner**

```verilog
**** bandgap reference circuit using self-biase current mirror *****

.lib "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** circuit definition ***

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-upcircuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	ra1	na1	vdd	sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1	q1	qp1	dc	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	net1	dc	0
vid5	net5	net6	dc	0

*** supply voltage
vsup	vdd	gnd	pulse	0	2	10n	1u	1u	1m	100u
.tran	5n	10u

.control
run

plot v(vdd) v(net1) v(net2) v(qp1) v(ra1) v(qp2) v(vref) v(qp3)
plot vid1#branch vid2#branch vid3#branch vid4#branch vid5#branch

.endc
.end
```

We can now simulate and plot a transient response of the curves for V<sub>REF</sub>, various node voltages and branch currents.

![bgr-comp-trans1](Day2/2-44.png)
![bgr-comp-trans2](Day2/2-45.png)
![bgr-comp-trans3](Day2/2-46.png)

Here, we can see that it takes approximately 1.2 &mu;sec for all the node voltages as well as V<sub>REF</sub> to build up to constant value.

### Detailed Plots for Start-up Simulation

To analyse the working of the start-up circuit in combination with the BGR circuit, let us run transient analysis as follows.

```verilog
**** bandgap reference circuit using self-biase current mirror by emad h. *****

.lib "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130.lib.spice tt"
.include "/home/srath22/cad/eda-technology/sky130/models/spice/models/sky130_fd_pr__model__pnp.model.spice"

.global vdd gnd
.temp 27

*** mosfet definitions self-biased current mirror and output branch
xmp1	net1	net2	vdd	vdd	sky130_fd_pr__pfet_01v8_lvt	l=2	w=5	m=4
xmp2    net2    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5    	m=4
xmp3    net3    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=4
xmn1    net1    net1    q1      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8
xmn2    net2    net1    q2      gnd     sky130_fd_pr__nfet_01v8_lvt     l=1     w=5     m=8

*** start-up circuit
xmp4    net4    net2    vdd     vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp5    net5    net2    net4    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=1
xmp6    net7    net6    net2    vdd     sky130_fd_pr__pfet_01v8_lvt     l=2     w=5     m=2
xmn3    net6    net6    net8    gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1
xmn4    net8    net8    gnd     gnd     sky130_fd_pr__nfet_01v8_lvt     l=7     w=1     m=1

*** bjt definition
xqp1	gnd	gnd	qp1	vdd	sky130_fd_pr__pnp_05v5_W3p40L3p40	m=1
xqp2    gnd     gnd     qp2     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=8
xqp3    gnd     gnd     qp3     vdd     sky130_fd_pr__pnp_05v5_W3p40L3p40       m=1

*** high-poly resistance definition
xra1	   ra1	    na1	    vdd	    sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra2   	na1     na2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra3    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xra4    na2     qp2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

xrb1    vref    nb1     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb2    nb1     nb2     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb3    nb2     nb3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb4    nb3     nb4     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb5    nb4     nb5     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb6    nb5     nb6     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb7    nb6     nb7     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb8    nb7     nb8     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb9    nb8     nb9     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb10   nb9     nb10    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb11   nb10    nb11    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb12   nb11    nb12    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb13   nb12    nb13    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb14   nb13    nb14    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb15   nb14    nb15    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb16   nb15    nb16    vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb17   nb16    qp3   	 vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8
xrb18   nb16    qp3     vdd     sky130_fd_pr__res_high_po_1p41     w=1.41  l=7.8

*** voltage source for current measurement
vid1    q1	     qp1	    dc     	0
vid2    q2      ra1     dc      0
vid3    net3    vref    dc      0
vid4    net7	   net1	   dc	     0
vid5	   net5	   net6	   dc	     0

*** supply voltage
vsup	vdd	gnd	pulse	0	2	10n	1u	1u	1m	100u
.tran	5n	10u

.control
run

.endc
.end
```

If we run a transient analysis, we should get the following curves.

![bgr-startup-vref](Day2/2-47.png)

We can observe that the ciruit has a start-up time of around 1.2 &mu;sec, which is well under the required 2 &mu;sec specification.

![bgr-startup-net12](Day2/2-48.png)

![bgr-startup-net62](Day2/2-49.png)

If we compare the voltage curves for net6 and net2, we can see that both nets follow V<sub>DD</sub>, but with some difference. When the voltage difference between them is greater than the threshold voltage of MP<sub>6</sub>, the current starts to flow through nets 1 and 2, and the BGR starts up.

![bgr-startup-vid4](Day2/2-50.png)

The graph above shows the current through the transistor MP<sub>6</sub>, and how it isolates itself from the BGR circuit after strat-up.

Let us look at what happens if we remove the start-up circuit. We have commented out the transistor MP<sub>6</sub> from the spice netlist. Below, are the plots after simulation.

![bgr-nostartup-v](Day2/2-51.png)
![bgr-nostartup-vnets](Day2/2-52.png)
![bgr-nostartup-vid](Day2/2-53.png)

Without the transistor MP<sub>6</sub>, we can see that net2 closely follows and remains at V<sub>DD</sub>, while net1 remains almost at GND. Also, the current in all branches is near zero. This shows the BGR circuit's dependence on a start-up circuit.

### Layout Design

We shall use the open source layout editor tool Magic. We can call the Magic GUI and console using the following command

![mag-cmd](Day2/2-3.png)

**Basic Cell Layouts**

**1.  Resistor**

![mag-res1](Day2/2-54.png)

**2.  PNP (BJT)**

![mag-bjt](Day2/2-55.png)

**3.  NFET**

![mag-NFET1](Day2/2-56.png)
![mag-NFET2](Day2/2-57.png)

**4.  PFET**

![mag-pfet](Day2/2-58.png)

**Block Layouts**

**1.  Resistor Bank**

![mag-resbank](Day2/2-59.png)

**2.  PFETs Block**

![mag-pfet-blk](Day2/2-60.png)

**3.  NFETs Block**

![mag-nfet-blk](Day2/2-61.png)

**3.  PNP10 (BJT) Block**

![mag-pnp-blk](Day2/2-62.png)

**4.  Starter NFET**

![mag-starter-nfet](Day2/2-63.png)

### Top Level Layout

![mag-top-layout](Day2/2-64.png)

### Top Level Layout Extraction, LVS and Post-Layout Simulation

We can extract the layout either cell by cell, or extract the complete layout netlist. To extract cell-by-cell, we first go to Options -> Cell Manager. We can now select the cell we want to load. This is shown below.

![mag-load-celmgr](Day2/2-65.png)

Next, we extract the NFET cells using the following commmands.

```
extract all
ext2sim label on
ext2sim
ext2spice scale off
ext2spice hieracrhy off
ext2spice
```
![mag-ext-cmd](Day2/2-66.png)

If we look at the nfets.spice file created by Magic, we can see the parasitics extracted at the bottom of the netlist, as shown below.

![mag-nfets-net](Day2/2-67.png)

Similarly, we must extract thespice files from Magic for all the remaining subcells (pnp10, pfets, startenfet and resbank). Now, we can extract the top level layout using the same commands as before. We should get the following extracted netlist.

> Note: For LVS in netgen, we must include the netlists for the 5 subcells. This is because the top level netlist does not include the cell definitions for the subcells. We must  also comment out all the parasitics in the top level netlist, as well as all the 5 lower level netlists for LVS, as the parasitics are devices in the netlist but aren't defined.

![mag-top-net](Day2/2-68.png)

Now, we can run the command `netgen` to call the Netgen console. Here, we type the following command to run LVS.

![netgen-lvs-cmd](Day2/2-69.png)

Once done, we should be able to see that both the schematic and layout netlists match uniquely.

![netgen-lvs-match](Day2/2-70.png)
![netgen-lvs-match-file](Day2/2-71.png)

Finally, we can do post layout simulations using the extracted and edited spice file. Here, we must add include the paths to the sky130.lib.spice and sky130_fd_pr__model__pnp.model.spice files; and can include the parasitics as well. We sahll use the ss corner for this simulation. If done correctly, we should get the following plot.

![postlayout-sim](Day2/2-72.png)

As we can see, even with the ss corner we get around 25 ppm/&deg;C in our post layout simulation. This is a major improvement over the pre-layout simulations using ss corner.

## Acknowledgements

- [Santunu Sarangi](https://www.linkedin.com/in/santunu-sarangi-b731305b)
- [Saroj Rout](https://www.linkedin.com/in/sroutk/)
- [Kunal Ghosh](https://github.com/kunalg123)
- [VSD-IAT](https://vsdiat.com/)


<!--- no startup specify and elaborate --->
