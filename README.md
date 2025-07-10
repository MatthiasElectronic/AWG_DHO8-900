Artificial Waveform Generator for DHO800 and DHO900 series
===

This is a daughter board for the Rigol DHO800 or DHO900 series oscilloscopes to add the missing hardware for the function generator.  
State: finished and working

[Download all files as zip archive here](https://github.com/MatthiasElectronic/AWG_DHO8-900/archive/refs/heads/main.zip)

Discussion thread: https://www.eevblog.com/forum/testgear/awg-function-generator-for-dho800-900-series-oscilloscopes/

![Top_View](https://github.com/user-attachments/assets/391ab191-d99d-4693-87ea-7b54f5daa720)
![bottom_view](https://github.com/user-attachments/assets/b3863e5f-22ea-4b8a-b6f5-83a7b127697f)
![Overview](https://github.com/user-attachments/assets/b83f5fdc-5962-46e9-9f2c-e5e7d89e1459)


Table of Contents
===
1. [Target of this project](#target-of-this-project)
2. [Versions](#versions)
3. [Performance](#performance)
4. [Requirements](#requirements)
5. [Cost and Ordering](#cost-and-ordering)
6. [Interface between Mainboard and AWG daughter board](#interface-between-mainboard-and-awg-daughter-board)
7. [Topology discussion and design decisions](#topology-discussion-and-design-decisions)
8. [Possible Improvements](#possible-improvements)
9. [Thanksgiving](#thanksgiving)
10. [Disclaimer](#disclaimer)

Target of this project
-------
1. This is NOT a copy of the original function generator, but it uses the same interface and same DAC.
2. This design is optimized for cheap production including assembly, by the minimizing the amount of "extended components" from the JLCPCB manufacturer.
3. Minimizing distortion (THD) and noise (SNR) is achieved.
4. The ADC output filter is designed for better pulse and step response, therby having worse anti-alias attenuation (for >10MHz signals) and slight passband signal attenuation
5. Gain and offset tolerances are worse. A significant contributor is anyhow on the oscilloscope mainboard, so perfection cannot be achieved. However, if unlucky, tolerances in this project can mathematically hit quite hard.
6. As the original AWG, this function generator is designed for a 50 OHm coax cable with high impedance termination.

Versions
-------
V1: Tested and working, but has an offset bug and smaller mechanical misaligments.  
V2: Recommended. Contains fixes from V1, but new Gerber files have not yet been tested/produced.  
More details are in the history page in the schematic.

Performance
---
All signals are created with this AWG board, mounted on a DHO804 with a DHO924S vendor.bin.  
Measured on V1 PCB with the modifications of the V2 schematic.  

### Switching edges of a rectangular waveform
The switching edges are clean. trise & tfall are 15.5ns. This is a picture of maximum amplitude (+/-5V), but rise and fall times were measured identical for smaller voltages.  
![Rect](https://github.com/user-attachments/assets/c2b512a7-d432-44c5-9338-02d2765cb7e3)

### Bandwidth
The -3dB point is slightly above 25MHz. At the maximum selectable frequency of 25MHz, a peak-to-peak voltage of 719mV is measured, which is just above the 707mV of the -3dB point.  
![Bandwidth](https://github.com/user-attachments/assets/b3b38e1e-0329-405f-9f28-7e2de31cba36)

### Distortion / THD
Distortion is over the most parts independent from the amplitude, just at very small amplitudes this is different.  
Here, a measurement of a 1kHz sine signal at 10V and 100mV peak-to-peak voltage is provided.  
At 10Vpp, the dominant HD3 is at ca. -62dB, so THD should be below -60dB.  
![Distortion_10V](https://github.com/user-attachments/assets/23c8edd6-0f0b-4b05-8764-f808a8116ff2)
![Distortion_100mV](https://github.com/user-attachments/assets/01ded008-5f99-4ae2-b609-c4911f7374a7)

### Noise / SNR
**Noise measurement setup:**  
A sine signal at very low frequency is applied, the trigger is set to the crossover and the horizontal range is small such that the signal appears as DC.  
→ This way, AC RMS can be measured and equals the RMS noise at that signal amplitude. However, ultra low frequency noise is not visible.

**Noise at 2Vpp signal amplitude:**  
![Noise_2V_Coax](https://github.com/user-attachments/assets/b128706f-0fe3-4c25-a284-ac9d7291351b)  
Noise RMS = 334µV, Signal RMS = 707mV => SNR = 67dB

**Noise at 10Vpp signal amplitude:**  
![Noise_AWG_On_10V](https://github.com/user-attachments/assets/dada7f82-e60a-4631-be19-1682378bc5aa)  
Noise RMS = 5.2mV, Signal RMS = 3.54V => SNR = 57dB

**Noise source**  
The measured noise levels are dominated by ripples/spikes from radiated EMI of the switching requlators on the mainboard with unshielded inductors.  
A 1.3MHz ripple is always present.  
A 740kHz ripple is only visible for >2Vpp, when the x10 gain stage (relay K4) is active..  
In the following FFT of a 10Vpp 1MHz sine signal, both ripples appear as peaks:  
![Ripple_740kHz](https://github.com/user-attachments/assets/d1d1495c-e259-4d74-b4fc-a288b3b6b7f0)  
Luckily, this is coupled into the circuit before the attenuators, so at smaller signal amplitudes also the EMI ripple is attenuated.

In addition, there are random (burst mode) spikes clearling coming from the PSU brick. They can even be measured with the AWG switched off.  
![Outside_Noise_Incoupling](https://github.com/user-attachments/assets/48269cfd-e4ea-487d-ac2f-68c659ffd7df)  

**To clarify: The noise/ripple is small enough, it is not visible in time domain unless having a very small amplitude.**

### Gain and Offset accuracy
My prototype has -3mV DC offset, coming directly from the motherboard on the target "DC offset" pin. This is the most noticable offset error, because this one is fully visible at the smallest signal amplitudes.  
Fixed this for my board, but this is completely related to tolerance. Additional offset errors are given in dependency of the AC gain and range selection, also this is amplified by 10 when the output signal exceeds 1Vp.  

My oscilloscope mainboard has a gain error of 3% between highest and lowest gain reference value. This is the most dominant part of the gain error, so if there is a need to fix this tolerance, this has to be done specific for each oscilloscope.

Requirements
--------
**Change vendor.bin**  
To add this AWG to the oscilloscope, a modified vendor.bin (DHO914S or DHO924S) to enable the AWG software functionalities is required.
https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/msg5344076/#msg5344076

**Offset Removal**  
After modifying the vendor.bin, a DC offset in the analog measurement channels is there and might even not be removable by recalibration.  
A firmware update AFTER changing the vendor.bin, followed by a recalibration helps. I assume that the FPGA fimware is changed during a firmware update in dependency of the vendor.bin.

**Hardware version resistors**  
Update: There is no need to change the resistors for version detection - both DHO800 and DHO900 series oscilloscopes work without changing resistors.

**Add missing components to the mainboard**  
The DHO800 series oscilloscopes are missing some parts on the mainboard.  
- The connectors (1.27mm female sockets) 2x20 and 2x5 have to be added (this might be helpful, thanks to Mechatrommer: https://www.youtube.com/watch?v=dBg6KQtSWTM)
- Two OP Amps (3PEAK TP1282L1-VR) have to be added for the offset and gain analog values. These are not really suited for this operation (because they are not officially input rail-rail), but somehow this still works and they are also placed on the original DHO900 series.
- The BNC connector is missing, but as this part is extremely high and nowhere to find, I instead increased the AWG PCB size and simply mounted a smaller BNC connector on the AWG board. So no extra BNC connector on the mainboard. When soldering the BNC connector onto the daughterboard, I suggest to do this as the last step. The SMD PCB interconnects have some tolerance and also the BNC connector ideally should not be pushed completely into the PCB to achieve the same height as the "trigger out" from the mainboard. I aligned the connector before soldering with the AWG daughterboard mounted on the mainboard.
- The board needs to be fixed to the mounting holes. M4 screws with two M4 fasteners were sufficient, but are slightly too large (soldermask IS the isolation...). A better solution (idea from hochohmig.de) are solderable M3 through hole taps, this is cleaner and does not require access to the backside of the mainboard. Securing the daughterboard to the mainboard with the mounting holes is absolutely necessary, the PCB interconnects alone are not sufficient as the BNC connector is not fastened to the case.
- A hole needs to be drilled in the plastic casing for the BNC connector.  
![unplaced](https://github.com/user-attachments/assets/92a66156-b65b-4f1a-aa15-0e5811540ead)
![OP_AMPs](https://github.com/user-attachments/assets/e35d6e20-c7d4-4be1-906d-9def2d9c173f)
![placed](https://github.com/user-attachments/assets/83820e1c-8084-4c5f-86da-6679e1be05f5)  
![BNC_inside](https://github.com/user-attachments/assets/3ee46990-70c8-4f25-ae4c-3945a1c385e8)
![BNC_outside](https://github.com/user-attachments/assets/8a6e7c0e-b36f-4f95-84f3-313098294ea1)
![through_hole_tap](https://github.com/user-attachments/assets/ed9987e9-c205-472e-9dcb-e8455fec63da)


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
CLK = Sample rate = 156MHz  
The digital data is ALWAYS completely using up the headroom of the DAC.  
Adjustments on the gain are only done by the analog gain input voltage, not digitally.  
With Amplitude = Vpp set to 2V, all attenuation/amplification stages K1-K4 are disabled and the analog gain input is at its maximum value.  
Still unclear is the "Protection" output signal, also it seems strange that this has to be pulled down to negative voltage.  
The 5 relays are: /sqrt(10), /10, /10, *10, On/Off

Topology discussion and design decisions
---
The topolology is  
![Topology_updated](https://github.com/user-attachments/assets/6244644b-ca57-4440-9b2e-0a566698f798)

The DAC itself is the same as on the original AWG board, but in a much cheaper QFN package. The allowed DAC output current was significantly overdriven in the original design, thus I reduced this current.

The OP amps are completely exchanged in favor for the price. The original 1GHz part just seemed to be overkill for a 50MHz signal generator. I chose the 230MHz OPA2673 instead. Sadly, there is nothing cheap in between that has high voltage and high bandwith. But the slew rate is sufficient.

Power rails are reduced to only +6.5V and -6.5V, which is just sufficient for the maximum +5V / -5V signal output voltage and at the border of the allowed OP amp operating conditions. These are both switchers, ripple is kept low by having forced PWM and filtering directly at the OP amp inputs. There is no need for LDOs as most LDOs cannot remove the 1.1MHz ripple. The 5V relays are rated for this voltage, so this is great as well.
Although the original board had +15V/-12V, the same maximum output signal is achieved. Having only 6.5V means that the DC offset stage had to be redone, therefore all PI attenuators and OP amp gains had to be changed. As an easy addon, I also added matched impedances on the PCB.

The DAC output filter is completly redone. The original filter required very specific and uncommon component values, which are nowhere availlable, so this had to be completely changed.

In the original design there was a 9th oder elliptic filter, which sounds great for HF suppression, but has a terrible step response. As I wanted solid edges for rectangular signals, I opted for a filter with a much better phase. While also optimizing for minimum component variation, this turned out to be a bit „creative“ and a 5th order lowpass filter (somewhere in between butterworth and bessel). Simulation in Spice looked promising, but the measured edges (see above, trise/fall = 15.5ns) are slightly slower than simulated (10ns).

As can be seen below, the step response is much better while not exceeding the OP ampl slew rate. Looking at the attenuation over frequency, obviously the elliptic filter is better. My filter has a slight attenuation in the upper passband and a not so perfect attenuation at the sample frequency (156MHz).

Attenuation in the passband: This is noticable for frequencies above 10MHz, but can be corrected for sine signals by manually adjusting the amplitude.

Aliasing: Aliasing can only occur above the Nyquist frequency (78MHz). For signals with lower frequencies (e. g. <1MHz), where fsig << fsample, this is not important as the sinc-behavior in the frequency domain of the sample and hold circuit does sufficently suppress any aliasing (sinc(1-1MHz/156MHz)=-44dB), in addition to the lowpass attenuation. For larger frequencies, this will be worse and probably noticable in the FFT or a spectrum viewer.

Conclusion:  
→ a worse aliasing attenuation, relevant only at high frequencies
→ a better behavior in the step and impulse response

![filter](https://github.com/user-attachments/assets/a4ed985c-6fe5-4654-b956-b0a9295e4c98)

Possible Improvements
---
...which I am not going to solve, but feel free to contribute.  
- the M4 fasteners are too large, more space is required as the solder mask is currently the only line of defence.  
- a DC offset potentiometer would be helpful  
- improve the gain accuracy, but this can get very difficult  
- choose connectors with through-hole alignment pins for easier assembly, but this might require better measurement of the dimensions.
- Eliminate the magnetic EMI incoupling of the switching regulators on the mainboard. Easiest solution could be swapping layer 3 (GND) and layer 4 (signal) and correcting trace widths for the impedances.
  However, as this is magnetic near field, this might not be sufficent. I tried some aluminum kitchen foil connected to GND as a shield, which brought -3dB. A floating thin iron sheet brought -10dB.
  Another option could be to replace the unshielded inductors on the main board with shielded ones.

Be warned that on the left hand side from the relays, the plastic case gets very close to the PCB, so even large MLCCs or SMD inductors can be too large and the relays definitly are.

Thanksgiving
---
Thanks to  
Everybody contributing to the [Hacking the Rigol DHO800/900 Scope](https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/) thread  
Pu6k1n [EEVBlog] for the [schematic of the reverse-engineered original AWG](https://www.eevblog.com/forum/testgear/hacking-the-rigol-dho800900-scope/?action=dlattach;attach=2492911), the pinning helped a lot  
AndyBig [EEVBlog] for all information about DHO800 to DHO900 vendor.bin modification  
Mechatrommer [EEVBlog] for inspiration from his own AWG attempt and lots of tiny information  
[hochohmig.de](https://www.hochohmig.de/) for providing calculation tools  
Contributers to KiCad and the library plugins

Disclaimer
---
This is published with CERN-OHL-W-2.0 license, which is accessible in license.txt.
No liability is provided for this not working or causing any form of damage or harm. This is not a product, but a development board. Use at your own risk.  
This project is influenced by the original AWG from Rigol, so only recommended for personal use. Although not checked, selling this AWG could possibly violate patents or rights of Rigol.

