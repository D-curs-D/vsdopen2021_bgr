# BGR_sky130
This github repository is for the design of a Band Gap Reference Circuit (BGR) using Google-skywater130 PDK.

## Introduction to BGR
The Bandgap Reference (BGR) is a circuit which provides a stable voltage output which is independent of factors like temperature, supply voltage. 
<p align="center">
  <img src="/Images/BGR1.png">
</p>


### Why BGR 
- A battery is unsuitable for use as a reference voltage source.
  - voltage drops over time
- A typical power supply is also not suitable 
  - noisy output and/or residual ripple.
- A voltage reference IC used buried Zener diode, 
  - Discrete design required additional components and high frequency filtering circuits due to higher thermal noise.
  - Low voltage Zener  diode is not available

**Solution**
- A Bangap reference which can be integrated in bulk CMOS, Bi-CMOS or Bipolar technologies without the use of  external components.

### Features of BGR
- Temp. independent voltage reference circuit widely used in Integrated Circuits
- Produces constant voltage regardless of power supply variation, temp. Changes and circuit loading
- Output voltage of 1.2v (close to the band gap energy of silicon at 0 deg kelvin)
- All applications starting from analog, digital, mixed mode, RF and system-on-chip (SoC).

### Applications of BGR
- Low dropout regulators (LDO)
- DC-to-DC buck converters
- Analog-to-Digital Converter (ADC)
- Digital-to-Analog Converter (DAC)



## Contents
- [1. Tool and PDK Setup](#1-Tools-and-PDK-setup)
  - [1.1 Tools Setup](#1.1-Tools-setup)
  - [1.2 PDK Setup](#1.2-PDK-setup)
- [2. BGR introduction](#2-BGR-introduction)
  - [2.1 BGR Principle](#2.1-BGR-Principle)
  - [2.2 Types of BGR](#2.2-Types-of-BGR)
  - [2.3 Self-biased Current Mirror based BGR](#2.3-Self-biased-current-mirror-based-bgr)
- [Design and Pre-layout Simulation](#Design-and-pre-layout-simulation)
- [Layout Design](#Layout-design)
- [LVS and Post-layout Simulation](#LVS-and-post-layout-simulation)


## 1. Tools and PDK setup

### 1.1 Tools setup
For the design and simulation of the BGR circuit we will need the following tools.
- Spice netlist simulation - [Ngspice]
- Layout Design and DRC - [Magic]
- LVS - [Netgen]

#### 1.1.1 Ngspice 
![image](https://user-images.githubusercontent.com/49194847/138070431-d95ce371-db3b-43a1-8dbe-fa85bff53625.png)

[Ngspice](http://ngspice.sourceforge.net/devel.html) is the open source spice simulator for electric and electronic circuits. Ngspice is an open project, there is no closed group of developers.

[Ngspice Reference Manual][NGSpiceMan]: Complete reference manual in HTML format.

**Steps to install Ngspice** - 
Open the terminal and type the following to install Ngspice
```
$  sudo apt-get install ngspice
```
#### 1.1.2 Magic
![image](https://user-images.githubusercontent.com/49194847/138071384-a2c83ba4-3f9c-431a-98da-72dc2bba38e7.png)

 [Magic](http://opencircuitdesign.com/magic/) is a VLSI layout tool.
 
**Steps to install Magic** - 
 Open the terminal and type the following to install Magic
```
$  wget http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz
$  tar xvfz magic-8.3.32.tgz
$  cd magic-8.3.28
$  ./configure
$  sudo make
$  sudo make install
```
#### 1.1.3 Netgen
![image](https://user-images.githubusercontent.com/49194847/138073573-a819cc67-7643-4ecf-983d-454d99ec5443.png)

[Netgen] is a tool for comparing netlists, a process known as LVS, which stands for "Layout vs. Schematic". This is an important step in the integrated circuit design flow, ensuring that the geometry that has been laid out matches the expected circuit.

**Steps to install Netgen** - Open the terminal and type the following to insatll Netgen.
```
$  git clone git://opencircuitdesign.com/netgen
$  cd netgen
$  ./configure
$  sudo make
$  sudo make install 
```
### 1.2 PDK setup

A process design kit (PDK) is a set of files used within the semiconductor industry to model a fabrication process for the design tools used to design an integrated circuit. The PDK is created by the foundry defining a certain technology variation for their processes. It is then passed to their customers to use in the design process.

The PDK we are going to use for this BGR is Google Skywater-130 (130 nm) PDK.
![image](https://user-images.githubusercontent.com/49194847/138075630-d1bdacac-d37b-45d3-88b5-80f118af37cd.png)

**Steps to download PDK** - Open the terminal and type the following to download sky130 PDK.
```
$  git clone https://github.com/RTimothyEdwards/open_pdks.git
$  cd open_pdks
$  ./configure [options]
$  make
$  [sudo] make install
```

## 2. BGR Introduction

### 2.1 BGR Principle
The operation principle of BGR circuits is to sum a voltage with negative temprature coefficient with another one exhibiting opposite temperature dependancies. Generally semiconductor diode behave as CTAT i.e. Complement to absolute temp. which means with increase in temp. the voltage across the diode will decrease. So we need to find a PTAT circuit which can cancel out the CTAT nature i.e. with rise in temp. the voltage across that device will increase and thus we can get a constant voltage reference with respect to temp.
<p align="center">
  <img src="/Images/BGR_Principle.png">
</p>

#### 2.1.1 CTAT Voltage Generation
Usually semiconductor diodes shows CTAT behaviour. If we consider constant current is flowing through a forwrard biased diode, then with increase in temp. we can observe that the voltage across the diode is decreaseing. Generally, it is found that the slope of the V~Temp is -2mV/deg Centigarde.
<p align="center">
  <img src="/Images/CTAT.png">
</p>

#### 2.1.2 PTAT Voltage Generation
<p align="center">
  <img src="/Images/Equation.png">
</p>

From Diode current equation we can find that it has two parts, i.e. 

- Vt (Thermal Voltage) which is directly proportional to the temp. (order ~ 1)
- Is (Reverse saturation current) which is directly proportional to the temp. (order ~ 2.5), as this Is term is in denominator so with increase in temp. the ln(Io/Is) decreases which is responsible for CTAT nature of the diode.

So to get a PTAT Voltage generation circuit we have to find some way such that we can get the Vt separated from Is.

To get Vt separated from Is we can approach in the following way
<p align="center">
  <img src="/Images/PTATCKT.png">
</p>

In the above circuit same amount of current I is flowing in both the branches. So the node voltage A and B are going to be same V. Now in the B branch if we substract V1 from V, we get Vt independent of Is.
<p align="center">
  <img src="/Images/PTATEQN.png">
</p>
Now

```
V= Combined Voltage across R1 and Q2 (CTAT in nature but less sloppy)
V1= Voltage across Q2 (CTAT in nature but more sloppy)
V-V1= Voltage across R1 (PTAT in nature)
```
From above we can see that the voltage V-V1 is PTAT in nature, but it's slope is very less as compared to the CTAT, so we have to increase the slope. In order to increase the slope we can use multiple BJTs as diode, so that current per individual diode will be less and it the slope of V-V1 will increase.
<p align="center">
  <img src="/Images/PTAT.png">
</p>

### 2.2 Types of BGR
Architecture wise BGR can be designed in two ways

- Using Self-biased current mirror  
- Using Operational-amplifier 

Application wise BGR can be categorized as
- Low-voltage BGR
- Low-power BGR
- High-PSRR and low-noise BGR
- Curvature compensated BGR

We are going to design our BGR circuit using Self-biased current mirror architecture.

### 2.3 Self-biased current mirror based BGR

The Self-biased current mirror based constitute of the following components.

- CTAT voltage generation circuit
- PTAT voltage generation circuit
- Self-biased current mirror circuit
- Reference branch circuit
- Start-up circuit

#### 2.3.1 CTAT Voltage generation circuit
The CTAT Voltage generation circuit consist of a BJT connected as a diode, which shows CTAT nature as explained above.
<p align="center">
  <img src="/Images/CTAT1.png">
</p>

#### 2.3.2 PTAT Voltage generation circuit
The PTAT Voltgae generation circuit consist of **N** BJTs connected with a series resistance. The operation principle is explained above.
<p align="center">
  <img src="/Images/PTAT1.png">
</p>

#### 2.3.3 Self-Biased Current Mirror Circuit
The Self-biased current mirror is a type of current mirror which requires no external biasing. This current mirrors biases it self to the desired current value without any external current source reference. 
<p align="center">
  <img src="/Images/currentmirror.png">
</p>

#### 2.3.4 Reference Branch Circuit
The reference circuit branch performs the addition of CTAT and PTAT volages and gives the final reference voltage. We are using a mirror transitor and a BJT as diode in the reference branch. By virtue of the mirror transistor in the reference branch the same amount of current flows through it as of the current mirror branches. Now from the PTAT circuit branch we are getting PTAT voltage and PTAT current. The same PTAT current is flowing in the reference branch. But the slope of PTAT voltage is much more smaller than that of slope of CTAT voltgae. In order to make increase the voltage slope we have to increase the resistance (current constant, so V increases with increase in R). Now across the high resistance we will get our constant reference voltage which is the result of CTAT Voltage + PTAT Voltage.
<p align="center">
  <img src="/Images/refbranch1.png">
</p>

#### 2.3.5 Start-up circuit
The start-up circuit is required to move out the self biased current mirror from degenerative bias point (zero current). The start-up circuit forecefully flows a slow amount of current through the self-biased current mirror when the current is 0 in the current mirror branches, as the current mirror is self biased this small current creats a disturbance and the current mirror auto biased to the desired current value.
<p align="center">
  <img src="/Images/startup.png">
</p>

#### 2.3.6 Complete BGR Circuit
Now by connecting all above components we can get the complete BGR circuit.
<p align="center">
  <img src="/Images/fullbgr.png">
</p>
Advantages of Self-biased BGR:

- Simplest topology
- Easy to design 
- Always stable

Limitations of Self-biased BGR:

- Low power supply rejection ratio (PSRR)
- Cacode design needed to reduce PSRR
- Voltage head-room issue
- Need start-up circuit

## 3. Design and Prelayout Simulation
For the real-time circuit design we are going to use sky130 technology PDK. Before we design the complete circuit we must know what are our design requirements. The design requirements are the design guidelines which our design must satisfy.

### 3.1 Design Requirements
- Supply voltage = 1.8V
- Temperature: -40 to 125 Deg Cent.
- Power Consumption < 60uW
- Off current < 2uA
- Start-up time < 2us
- Tempco. Of Vref < 50 ppm

Now, we have to go through the device data sheet to find the appropriate devices for our design. 

After thoroughly going through the device data sheet we selected the following devices for our design.
### 3.2 Device Data Sheet
***1. MOSFET***
| Parameter | NFET | PFET |
| :-: | :-: | :-: |
| **Type** | LVT | LVT |
| **Voltage** | 1.8V | 1.8V |
| **Vt0** | ~0.4V | ~-0.6V |
| **Model** | sky130_fd_pr__nfet_01v8_lvt | sky130_fd_pr__pfet_01v8_lvt |

***2. BJT (PNP)***
| Parameter | PNP | 
| :-: | :-: | 
| **Current Rating** | 1uA-10uA/um2 | 
| **Beta** | ~12 |
| **Vt0** | 11.56 um2 | 
| **Model** | sky130_fd_pr__pnp_05v5_W3p40L3p40 |

***3. RESISTOR (RPOLYH)***
| Parameter | RPOLYH | 
| :-: | :-: | 
| **Sheet Resistance** | ~350 Ohm | 
| **Tempco.** | 2.5 Ohm/Deg Cent |
| **Bin Width** | 0.35u, 0.69u, 1.41u, 5.37u | 
| **Model** | sky130_fd_pr__res_high_po |







[Magic]:                http://opencircuitdesign.com/magic/
[Ngspice]:              http://ngspice.sourceforge.net
[Netgen]:               http://opencircuitdesign.com/netgen/
[NGSpiceMan]:           http://ngspice.sourceforge.net/docs/ngspice-html-manual/manual.xhtml
