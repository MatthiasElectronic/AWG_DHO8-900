# Artificial Waveform Generator for DHO800 and DHO900 series

This is a daughter board for the Rigol DHO800 or DHO900 series oscilloscopes to add the missing hardware for the function generator.  
State: finished and working

Discussion thread: https://www.eevblog.com/forum/testgear/awg-function-generator-for-dho800-900-series-oscilloscopes/

![Top_View](https://github.com/user-attachments/assets/391ab191-d99d-4693-87ea-7b54f5daa720)

Target of this project
-------
1. This is NOT a copy of the original function generator, but it uses the same interface and same DAC.
2. This design is optimized for cheap production including assembly, by the minimizing the amount of "extended components" from the JLCPCB manufacturer.
3. Minimizing distortion (THD) and noise (SNR) is achieved.
4. The ADC output filter is designed for better pulse and step response, therby having worse anti-alias attenuation (for >10MHz signals) and slight passband signal attenuation
5. Gain and offset tolerances are worse. A significant contributor is anyhow on the oscilloscope mainboard, so perfection cannot be achieved. However, if unlucky, tolerances in this project can mathematically hit quite hard.

Versions
-------
V1: Tested and working, but has an offset bug and smaller mechanical misaligments.  
V2: Recommended. Contains fixes from V1, but new Gerber files have not yet been tested/produced. 

Performance
---
All signals are created with this AWG board, mounted on a DHO804 with a DHO924S vendor.bin.  
DC and RMS levels are measured with a multimeter.

![performance](https://github.com/user-attachments/assets/591ed406-e5bc-40c9-9020-f47f37d0bc8e)

Here, a 1Vpp, 1kHz sine signal is displayed. Using the oscilloscopes own FFT: THD is perfect, harmonics are below the noise floor. The measureable SNR is somewhere around 80dB. Considering that this is an oscilloscope with a 12 Bit ADC, this shows that this is a quantization limitation of the analog input stage. So no major flaws in the design.

There is some noise transferred from the mainboard/PSU stage to the AWG board GND, even when the AWG (including the step down converters) is switched off.
Noise added by the AWG is not noticable. This noise is definitly relevant in the lowest signal range (2mVpp).

Gain and offset are not perfect.  
Offset: -3mV DC offset, coming directly from the motherboard. This is the most noticable offset error. Fixed this for my board, but this is completely related to tolerance. Additional offset errors are given in dependency of the AC gain and range selection, also this is amplified by 10 when the output signal exceeds 1Vp.  
Gain: From the mainboard an error of 3% between highest and lowest reference value for the gain is coming.

Also, the bandwidth limitation of the filter is noticable, the amplitude does drop at 50MHz.

The performance evaluation is still work in progress, therefore some values are missing.

Requirements
--------
**Change vendor.bin**  
To add this AWG to the oscilloscope, a modified vendor.bin (DHO914S or DHO924S) to enable the AWG software functionalities is required.
https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/msg5344076/#msg5344076

**Offset Removal**  
After modifying the vendor.bin, a DC offset in the analog measurement channels is there and might even not be removable by recalibration.  
A firmware update AFTER changing the vendor.bin, followed by a recalibration helps. I assume that the FPGA fimware is changed during a firmware update in dependency of the vendor.bin.

**Hardware version resistors**  
The hardware version configuration resistors possibly have to be changed.
To change the hardware version to a DHO900, one resistor on the bottom of the PCB has to be moved.  
DHO800 has hardware version 12 (0b1100), DHO900 has hardware version 8 (0b1000).  
https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/msg5389958/#msg5389958  
This step requires a recalibration afterwards. 
Don't think that this is really necessary, please give feedback.

After modification of the vendor.bin and the version resistors, this can be checked:  
![versions](https://github.com/user-attachments/assets/e568bba0-863e-49a2-b307-539e91a3b76c)

**Add missing components to the mainboard**  
- The connectors (1.27mm female sockets) 2x20 and 2x5 have to be added (this might be helpful, thanks to Mechatrommer: https://www.youtube.com/watch?v=dBg6KQtSWTM)
- Two OP Amps (3PEAK TP1282L1-VR) have to be added for the offset and gain analog values. These are not really suited for this operation (because they are not officially input rail-rail), but somehow this still works and they are also placed on the original DHO900 series.
- The BNC connector is missing, but as this part is extremely high and nowhere to find, I instead increased the AWG PCB and simply mounted a smaller BNC connector on the AWG board. So this can be left empty.
- The board need to be fixed to the mounting holes. M4 screws with two M4 fasteners were sufficient, but are slightly too large (soldermask IS the isolation...). Maybe M3 would be better.
- A hole needs to be drilled in the plastic casing for the BNC connector.  
![unplaced](https://github.com/user-attachments/assets/92a66156-b65b-4f1a-aa15-0e5811540ead)
![OP_AMPs](https://github.com/user-attachments/assets/e35d6e20-c7d4-4be1-906d-9def2d9c173f)
![placed](https://github.com/user-attachments/assets/83820e1c-8084-4c5f-86da-6679e1be05f5)

Luckily, all remaining components (resistors, reference voltages, digital interface, …) are already assembled.

**It is NOT necessary to add any additional RAM to the DHO800 series.**

Cost and Ordering
-------
Total cost for 1 board should be ca. 50€.

All components (apart from the connectors) are soldered on the top layer. The PCB is a four layer board.  
This allowed economic assembly with JLCPCB (not sponsored), which is very cheap when using the preferred components. So the whole design is optimized around these preferred components to keep cost low. However, there are some components that are extended (and this costs ca. 3€ each extra). So most of them were ordered on LCSC, because this order was anyways necessary for the connectors and the OP-Amps on the main board.
All parts in this design files have a LCSC number, which makes ordering at LCSC or JLCPCB very simple.
Just be sure that you
1. Have a hot air station or let the QFN packages be soldered (DAC & OP-Amp). Even when ignoring the bottom central GND-pad, this won’t be easy with a simple soldering iron.
2. Use a four layer bord with the recommended or a similar stackup as documented on the first page of the schematic to have somewhat correct impedances.
3. Have accessable vias such that you can use them as measurement points if something does not work.
4. Do not choose a different inductor with a larger height for the step down converters, as there is no space to the casing.

When keeping extended components low (I only chose the SMD inductor), then this can be quite cheap. I ended up with 34€ on JLCPCB for 5 partly assembled boards and 41€ on LCSC for components for 2 boards, including tax and shipment. This adds up to 75€. It should be ca. 53€ if I only ordered components for 1 board. (05/2025)

Interface between Mainboard and AWG daughter board
----
CLK = Sample rate = 256MHz  
The digital data is ALWAYS completely using up the headroom of the DAC.  
Adjustments on the gain are only done by the analog gain input voltage, not digitally.  
With Amplitude = Vpp set to 2V, all attenuation/amplification stages K1-K4 are disabled and the analog gain input is at its maximum value.  
Still unclear is the "Protection" output signal, also it seems strange that this has to be pulled down to negative voltage.  
The 5 relays are: /sqrt(10), /10, /10, *10, On/Off

Topology discussion and design decisions
---
The DAC itself is the same as on the original AWG board, but in a much cheaper QFN package. The allowed DAC output current was significantly overdriven in the original design, thus I reduced this current.

The OP amps are completely exchanged in favor for the price. The original 1GHz part just seemed to be overkill for a 50MHz signal generator. I chose the 230MHz OPA2673 instead. Sadly, there is nothing cheap in between that has high voltage and high bandwith. But the slew rate is sufficient.

Power rails are reduced to only +6.5V and -6.5V, which is just sufficient for the maximum +5V / -5V signal output voltage and at the border of the allowed OP amp operating conditions. These are both switchers, ripple is kept low by having forced PWM and filtering directly at the OP amp inputs. There is no need for LDOs as most LDOs cannot remove the 1.1MHz ripple. The 5V relays are rated for this voltage, so this is great as well.
Although the original board had +15V/-12V, the same maximum output signal is achieved. Having only 6.5V means that the DC offset stage had to be redone, therefore all PI attenuators and OP amp gains had to be redone. Thus, I also added matched impedances on the PCB.

The DAC output filter is changed. In the original design there was a 9th oder elliptic filter, which sounds great for HF suppression, but has a terrible step response. As I wanted solid edges for rectangular signals, I opted for a filter with a much better phase. While also optimizing for minimum component variation, this turned out to be a bit „creative“ and a 5th order lowpass filter (somewhere in between butterworth and bessel). But simulation in Spice looked promising and I measured the exact same step response as simulated.

As can be seen below, the step response is much better while not exceeding the OP ampl slew rate. Looking at the attenuation over frequency, obviously the elliptic filter is better. My filter has a slight attenuation in the upper passband and a not so perfect attenuation at the Nyquist frequency (128MHz). However, for signals in the <10MHz range, aliasing occurs close to the sampling frequency of 256MHz, where the attenuation is comparable to the elliptic filter.  
→ My filter has a worse aliasing attenuation at high frequencies, but a better behavior for step or impulse responses.  
→ Also, the original filter required very specific component values, which are not widely availlable. So I had no other chance than changing the filter.  
![filter](https://github.com/user-attachments/assets/a4ed985c-6fe5-4654-b956-b0a9295e4c98)

Possible Improvements
---
...which I am not going to solve, but feel free to contribute.  
- the M4 fasteners are too large, more space is required as the solder mask is currently the only line of defence.  
- a DC offset potentiometer would be helpful  
- improve the gain accuracy, but this can get very difficult  
- choose THR connectors for easier assembly, but this requires better measurement of the dimensions.

Be warned that on the left hand side from the relays, the plastic case gets very close to the PCB, so even large MLCCs or SMD inductors can be too large and the relays definitly are.

Thanks to
---
Everybody contributing to the [Hacking the Rigol DHO800/900 Scope](https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/) thread  
Pu6k1n [EEVBlog] for the [schematic of the reverse-engineered original AWG](https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/?action=dlattach;attach=2492911), the pinning helped a lot  
AndyBig [EEVBlog] for all information about DHO800 to DHO900 vendor.bin modification  
Mechatrommer [EEVBlog] for inspiration from his own AWG attempt and lots of tiny information  
[hochohmig.de](https://www.hochohmig.de/) for providing calculation tools  
Contributers to KiCad and the library plugins

Disclaimer
---
No liability is provided if this does not work or causes any form damage or harm. This is not a product, but a development board. Use at your own risk.
This is published with CERN-OHL-W-2.0 license, which is availlable as license.txt.
