# Table of Content
1. [Prologue](#prologue)
2. [Solution Space](#solution-space)
  1. [Linear Regulators](#linear-regulators)
  2. [Switching Mode Power Supplies (SMPS)](#switching-mode-power-supplies-smps)
    1. [Micro-power SMPS](#micro-power-smps)
3. [Current Design](#current-design)
  1. [Version History](#version-history)
  2. [Layout Software Used](#layout-software-used)
4. [License](#license)

---

# Prologue
It all started when I first met ESP8266, a tiny Wifi SoC with amazing versatility and potentials.

I want to put it everywhere in my house, gathering temperature, humidity, presence of human (PIR), positiong of windows and doors, ...

But I don't want to run 100 more wires in my house. I have to rely on batteries.

Batteries have physical dimentions, have weights, have capacity limits. And the good ones are expensive.

At the moment (2015~2016), LiPo 18650 seems to have the best combination of power density (capacity/volume) and price.
But, their voltage is too high at full charge (4.2v), but drop too low (2.7v) before they completely discharge. 

So I need regulators. Regulators have power consumptions, and many are quite heavy energy consumers themselves.
Suppose 3400mAh @ 3.3v of energy could last an ESP 6 month, add a switching regulator with I<sub>q</sub> (idle current use) of 2mA (actually not a very bad number, most ones you get from eBay or Amazon uses more than that), now that energy would be gone in a little over a month.

I can't imagine climbing ladders every week to change batteries everywhere in my house. That is crazy.

I need efficient power supplies, very efficient.

---

# Solution Space
## Linear Regulators
A simple and more efficient solution than lousy switching regulators is the low I<sub>q</sub>, low drop-out (LDO) linear voltage regulator.

A linear regulator functions like an adaptive variable resistor -- their internal resistance are automatically adjusted according to load, so that the voltage drop on the load is always a constant V<sub>Load</sub>, as long as the input voltage is higher than V<sub>Load</sub> + V<sub>Drop</sub>. Due to the nature of how they work, linear regulators require very few components, and modern ones can achieve very low V<sub>Drop</sub> and I<sub>q</sub>. For example, MPC1700 has V<sub>Drop</sub>=0.2v and I<sub>q</sub>=2uA.

But linear regulators have two weak spots, low conversion efficiency and V<sub>In</sub> > V<sub>Out</sub>.

1. Because regulation is done via equivalent resistance, effectively the energy carried by excess voltage is directly converted into heat. For example, when V<sub>In</sub>=4.2v and V<sub>Out</sub>=3.3v, the efficiency of linear regulation is only 3.3/4.2 = 78.6%

2. Linear regulation cannot boost voltage, and they always have a positive V<sub>Drop</sub>, so the usable voltage range of the battery is also narrowed. For example, with V<sub>Drop</sub>=0.2v, when V<sub>In</sub>=3.3v, V<sub>Out</sub> can only reach 3.1v.

## Switching Mode Power Supplies (SMPS)
SMPS is very different from linear regulators. They function by turning on and off the switch in the power supply circuit, at extremely high frequency, million times per second. Imagine a water dam, with water levels representing voltages. When the flood gate is open, water flow through, raising the water level of downstream; when the gate closes, water in the downstream drains and the level lowers. By carefully controlling the frequency and duty cycle of the gate, we can control the downstream water to a near constant level. In addition, by adding a pump at the flood gate, the downstream can even maintain higher water level than the dam reservoir!

Mechanism wise, SMPS is much more efficient than linear regulation, because excess energy is never discarded like in linear regulation. However, real world is never as simple. Switching on/off the gate has costs, the gate is also not completely water-tight, and there is also the cost of running the pump at the gate... all these auxiliary costs consumes the water in the dam, or the I<sub>q</sub> of the SMPS.

Macro-power SMPS have quite high I<sub>q</sub>, usually single or double digit mA. They are designed to handle high load current efficiently, e.g. 3~10A. When using those for micro-power applications, e.g. <1mA average current, they a grossly inefficient. Imagine a river dam that consumes about 1 ton of water each day, nobody cares about the consumption when 1 tons of water flow in every second; but when you give the dam operator a bottle water and say: "here, regulate this." guess what look will on his/her face...

### Micro-power SMPS
Micro-power SMPS are more dedicate versions of SMPS, which are designed to handle very small loads. Because the targeted load is small, auxiliary costs are also well-controlled. For example, TPS630252 is a 3.3v fixed voltage micro-power SMPS, with I<sub>q</sub> of 35~82uA, hundreds of times lower than some macro-power SMPS.

However, there is still a performance gap compared with 2uA I<sub>q</sub> of MCP1700. And this gap may become significant at extremely low load, such as 3.2uA deep sleep current of esp8266 (fully assembled modules use more power, around 100uA).

---

# Current Design
My design aims to reduce or eliminate this gap, by combining micro-power SMPS with very efficient power supervisory circuits. The logic of my design is that, at extremely low power draw, even the supply capacity of micro-power SMPS is humongous – power generated by 0.1s of working time is probably sufficient for 1~2s of power consumption, which means that, the SMPS is idle waiting most of the time. If we can completely shut down the SMPS for those periods, we could save much more energy.

TPS630252 has a shutdown mode, with current draw of only 0.1uA; the power supervisory chip, MC33464, has a constant current draw of around 1~2uA; supporting circuit adds about 1~2uA of additional current draw. Overall, the circuit should consume about 3~5uA at ultra-light load.

## Version History
V1 has a design flaw, which was discovered soon after drawing.
When Vin is higher than Vout, Vgs will be negative regardless of the !RST signal, thus renders the shutdown control ineffective.

V2 is also flawed, due to the failure to consider shutdown discharge circuit.
It was discovered on protoype PCB, a patch was made using diode, which confirmed the expected operation with the flaw corrected.

V3 is currently pending validation. Prototype PCB has been ordered, eta. Mid-March 2016.

## Layout Software Used
PCBWeb http://www.pcbweb.com/

---

# License
The circuit design and PCB layout is released under GPL license.
Proprietary licensed release is available upon request.

