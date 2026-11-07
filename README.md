# data-acquisition-boards
The **data acquisition** (or sensor) boards on the vehicle is responsible for powering and protecting sensors, filtering analog and digital telemetry data, and converting it to CAN to be logged. It is non-car-critical, meaning nothing on the board should be required for the vehicle to drive.
<br><br>
2025-2026 Sensor Board:
<br><br>
<img width="1750" height="899" alt="image" src="https://github.com/user-attachments/assets/e52823dc-6c6c-4c70-b1c8-5be231561d5b" />
<br><br>
Until recently, the vehicle ran a 4-board architecture (now dual boards).
<br><br>
<img width="1357" height="443" alt="image" src="https://github.com/user-attachments/assets/3a2761fd-017d-4c27-89c3-4678b08ef2e7" />
<img width="1357" height="449" alt="image" src="https://github.com/user-attachments/assets/62acb568-9ac4-45d1-8ae0-274b8416b59c" />
<br><br>
With consolidated front and rear boards, most things have to change, but the board structure remains the same:
<br><br>
<img width="1650" height="953" alt="image" src="https://github.com/user-attachments/assets/04cbe901-36c6-4b56-98af-bc135628c3cb" />
<br><br>
I/O is offboarded in the diagram.
<br><br>
**Power stage:**
<br><br>
<img width="1747" height="900" alt="image" src="https://github.com/user-attachments/assets/86c87a21-a7c8-4327-91e4-cc4a089916f1" />
<br><br>
Offboarded power goes to sensors, and on-board power is used for internal components such as the STM32H7 MCU used for processing telemetry data.
<br><br>
**Sensor power OCP:**
<br><br>
Rules require that the sensor boards provide over-current protection for power rails connecting the sensors. This was especially relevant with the 4-board architecture, as a fault with one board could short all 4 boards connected on the same rail.
<br><br>
For the 2025-2026 season, I ran trade studies on potential OCP components such as PTC and automotive fusing, e-fusing, load-switch ICs, ORing controllers, and op-amps.
<br><br>

| Buffer | Pros / Cons | Cost | Feasibility | Functionality | Time | Reliability | Packaging |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Integrated load-switch IC | **Pros:** Provides inrush current protection; takes up little space; automotive rating; cheap.<br>**Cons:** No reverse-battery protection; adds some resistance to the circuit (relevance uncertain). | TI load switches with short-circuit protection range from $0.061 to $0.271 per unit at 1,000 units. [TPS22950-Q1](https://www.ti.com/product/TPS22950-Q1) and [TPS22919-Q1](https://www.ti.com/product/TPS22919-Q1). | Basic current limiting; weaker protection and slower trip time than an eFuse. Easy to implement on a 5 V power rail with proper thermal layout. | The MCU can turn each sensor’s power rail on or off. During a short, the device limits current and heats up until thermal shutdown turns the switch off. The switch remains off until the die cools down. | Little effort or time to install. | Current limiting is less precise and slower than with an eFuse. Each thermal shutdown cycle shortens device life. | 2.9 mm × 2.8 mm to 2 mm × 2.1 mm for the TI parts listed. |
| P-channel MOSFET high-side switch | **Pros:**<br>**Cons:** | | | [Single P-Channel Enhancement-Mode MOSFETs datasheet (Rev. C)](https://www.ti.com/lit/ds/symlink/tps1101.pdf)<br>[High-Side MOSFET Input Switch Options — DigiKey](https://www.digikey.com/en/articles/high-side-mosfet-input-switch-options) | | | |
| FET ORing controllers | **Pros:** Likely more efficient for low-voltage applications.<br>**Cons:** Complex; can be expensive; does not inherently prevent shorting. | | | An IC and a MOSFET help isolate sources that fail or short. For ORing, a MOSFET is more useful when the diode’s forward voltage exceeds the full-load current multiplied by the MOSFET’s on-resistance. | | | |
| eFuses | **Pros:** More precise trip point; faster response; more reliable long term; smaller.<br>**Cons:** Significantly more expensive; run hot; take more time to install. | TI eFuses cost $2–$4 per unit at 1,000 units. [TI eFuses and hot-swap controllers](https://www.ti.com/product-category/power-management/power-switches/efuse-hot-swap-controllers/products.html). | eFuse ICs take up more space, release more heat, and require extra capacitors and resistors. Good thermal layout is needed to avoid hot spots. | eFuses are resettable, “more accurate,” “faster,” and can “repair themselves without user intervention” (Texas Instruments). Response is much faster than with polyfuses, sometimes under 100 ns. [EEPower](https://eepower.com/technical-articles/proof-of-concept-a-high-voltage-efuse-for-EV-apps) | Takes more time to install on the PCB than a polyfuse. Thermal or space constraints may require a board redesign. | Higher-grade automotive parts are accurate, reliable, and repeatable. Should not degrade. | Generally smaller than polyfuses, though the board may still require a redesign. |
| PTC / Polyfuses | **Pros:** Significantly cheaper; easier and faster to install; do not generate heat\*.<br>**Cons:** \*Sensitive to ambient temperature; less predictable trip point; unreliable long-term resistance and precision; potentially larger than eFuses; automatic reset needs testing. | Bel Fuse PTCs cost $0.06–$0.30 per unit and are sold in any quantity. [PTC Resettable Fuses — DigiKey](https://www.digikey.com/en/products/filter/ptc-resettable-fuses/150). | Easy to install with no extra components. Account for temperature sensitivity in the layout. | Automatically resets as it cools. Reset time needs testing. The trip point is less predictable, and resistance drifts with age and ambient temperature. Temperature sensitivity may require derating. Response time is slower at 20–1,000 ms. [Wayback Machine](https://web.archive.org/web/20150122051210/http://www.te.com/content/dam/te/global/english/products/Circuit-Protection/knowledge-center/documents/polyswitch-fundamentals.pdf) | Minimal effort and time; installation is similar to installing a resistor. | Long-term resistance and trip point may become unreliable. May need occasional replacement. | Lower-current polyfuses are smaller; higher-current polyfuses are larger. At high currents, they are much larger than eFuses. |
| Op Amp | **Pros:** High input impedance; low output impedance; unity gain.<br>**Cons:** Power consumption; can be complex; specific voltage ranges. | TI op-amps cost approximately $1–$10 depending on quantity and purpose. Some cost up to about $20. | The power supply limits the output. Temperature changes can cause drift. | Op-amps typically provide current gain but not voltage gain in the unity-gain configuration described. | Little effort or time to install. | May be unreliable at high currents; testing is needed. Can be paired with transistors, ICs, or other elements, which also requires testing. | Approximately 0.58 mm × 0.88 mm to 96 mm × 64 mm × 41 mm, depending on usage. |

<br><br>
Ultimately, we decided on PTC fusing with an op-amp buffer as our protection circuit. The PTC fusing provides OCP while the op-amp provides voltage buffering. This is the OCP circuit for the 3V3D rail:
<br><br>
<img width="1917" height="375" alt="image" src="https://github.com/user-attachments/assets/111310ce-256a-4d1a-9b82-190f8ba20e47" />
<img width="1035" height="533" alt="image" src="https://github.com/user-attachments/assets/ce8222dc-d8bc-4d78-8b67-22b13bcf412c" />
<br><br>
Unfortunately, this design had a critical fault: the op-amp was sinking significant current during normal use and would overheat, causing it to break. Upon further inspection, the PTC fuse never tripped, meaning most of if not all current was diverted to the op-amp. Essentially, the op-amp acted as a fuse. Not good.
<br><br>
For the 2026-2027 season, this is the primary issue we sought to resolve. We re-ran component research, prioritizing low power consumption, simplicity of implementation, and physical size. We brought back candidates from last year, reconsidering them for this year's board.
<br><br>

| Buffer | Pros | Cons | Cost | Feasibility | Functionality | Reliability / Reusability | Packaging |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Op-amp (current power buffer) | Works as a current buffer. | Runs very hot. | $1.530 per unit. [TI TLV9354-Q1](https://www.ti.com/product/TLV9354-Q1/part-details/TLV9354QPWRQ1) | Works in the current design. | Buffers current. | Reliable and reusable, with some heat issues. | TSSOP (PW) |
| eFuses | More precise trip point.<br>Faster response.<br>More reliable long term.<br>Smaller than polyfuses on average. | Significantly more expensive.<br>Run hot.<br>Take more time to install. | $2–$4 per unit at 1,000 units. [TI eFuses and hot-swap controllers](https://www.ti.com/product-category/power-management/power-switches/efuse-hot-swap-controllers/products.html) | The IC and additional capacitors and resistors take up board space. Heat dissipation requires careful layout to avoid hot spots. | Resettable. TI describes eFuses as “more accurate,” “faster,” and able to “repair themselves without user intervention.” Some respond in under 100 ns, much faster than polyfuses. [EEPower](https://eepower.com/technical-articles/proof-of-concept-a-high-voltage-efuse-for-EV-apps) | Higher-grade automotive parts are accurate, reliable, and repeatable. Performance is not expected to degrade. | Generally smaller than polyfuses. |
| PTC / Polyfuses | Significantly cheaper.<br>Easier and faster to install.<br>Generate less heat. | Sensitive to ambient temperature.<br>Less predictable trip point.<br>Resistance and precision can drift over time.<br>Potentially larger than eFuses.<br>Automatic reset needs testing. | Bel Fuse PTCs cost $0.06–$0.30 per unit and are sold in any quantity. [DigiKey PTC resettable fuses](https://www.digikey.com/en/products/filter/ptc-resettable-fuses/150) | Easy to install with no extra components. Layout must account for temperature sensitivity. | Automatically resets as it cools. Reset time needs testing. Resistance varies with age and ambient temperature, and temperature sensitivity may require derating. Trip response is slower at 20–1,000 ms. [PolySwitch Fundamentals (archived)](https://web.archive.org/web/20150122051210/http://www.te.com/content/dam/te/global/english/products/Circuit-Protection/knowledge-center/documents/polyswitch-fundamentals.pdf) | Resistance and trip point may become unreliable over time. Occasional replacement may be necessary. | Package size increases with rated current. At high currents, polyfuses are much larger than eFuses. |
| Ideal diode controllers (ORing controllers) | Low forward voltage drop, higher efficiency, and less heat.<br>Rapid reverse-current blocking.<br>Overvoltage and polarity-fault protection. | Higher cost.<br>Requires MOSFET selection.<br>Increases circuit complexity. | Automotive ORing controllers range from $0.965 to $2.277 per unit at 1,000 units. [TI product selection](https://www.ti.com/product-category/power-management/power-protection-switches-controllers/ideal-diode-oring-controllers/products.html#339=External%20back-to-back%20FET&480=1&1498=Automotive&1694=Ideal%20diode%20controller&238max=45%3B65&) | Selecting an external MOSFET adds design complexity. The circuit is robust, emits less heat than most options, and adds reverse-polarity protection during testing. | Drives an external N-channel MOSFET to pass forward current with minimal voltage drop and block reverse current. Automotive models can also provide overvoltage, overcurrent, and short-circuit protection. [TI controller overview](https://www.ti.com/product-category/power-management/power-protection-switches-controllers/ideal-diode-oring-controllers/overview.html) | Expected to be reliable over long-term use barring extreme conditions. | VQFN |
| Load-switch IC ([TPS22950-Q1](https://www.ti.com/product/TPS22950-Q1)) | Lower cost.<br>Easy to implement.<br>Multiple protection features. | Large footprint.<br>Lacks configurable trip timing and current-sense telemetry. | $0.067–$0.614 per unit at 1,000 units. [TI load-switch product selection](https://www.ti.com/product-category/power-management/load-switches/products.html#2282=Adjustable%3BFixed%3BShort%20circuit%20protection&) | Quick to implement with no extra components needed. | Inrush current control, quick output discharge, reverse-current protection, short-circuit protection, and thermal shutdown. [TI load-switch overview](https://www.ti.com/product-category/power-management/load-switches/overview.html) | Suitable for rail sequencing and moderate protection. Likely reliable for long-term automotive use. | Footprints range from under 1 mm² to over 5 mm². Automotive-grade package listed as SOT-23-THN (8.17 mm²). |

<br><br>
Many components were seriously considered for the final design. ORing controllers were an initial choice, but ultimately discounted as they were relatively complex to implement, required external components, and were not designed specifically for the function that we wanted it to perform (only as a side benefit).
<br><br>
Load-switch ICs were a promising choice; initially discounted due to a low voltage input range, upon verification with sensor voltage requirements we deemed it sufficient for our use case. However, these were new components not used elsewhere on the car, adding complexity and risk to implementation, and we found components better suited for our use case later on.
<br><br>
The final decision was between current-sensing e-fusing, automotive fusing, and PTC fusing. In short, current-sensing e-fusing uses shunt resistors and amplifiers to measure voltage drop across a resistor, calculating current. It samples voltages based off software settings and remotely switches a high-side driver (e-fuse) whenever it detects over-current. Software integration means extremely fast reaction time, adjustable over-current limiting and extra testing data. However, it is expensive, requires extra effort both with hardware and firmware, and takes up extra space on the board. While an extremely effective option, it is overkill for *most* of our sensor powering OCP applications.
<br><br>
Automotive fusing is the simplest option. It is a standard fuse that melts over a certain current threshold and requires replacement with every break. It is cheap, small, and fast. The only downside is that it needs replacement, but ideally the power rails should not short often and so the extra work would be negligible in the long run.
<br><br>
PTC fusing stands for positive temperature coefficient. The fuses are made of a material that expand at a certain temperature, increasing resistance massively. When it cools down, it conducts again, acting as a normal wire. We used this for the 2025-2026 sensor board OCP circuitry, and while it was not the main source of issues we had that year, some other concerns with this option were that the temperature threshold was not exact, and that the reaction time would take too long (much slower reaction time than both automotive fusing and e-fusing). However, we confirmed with the rest of the circuit design team that PTC fusing had a sufficiently low reaction time and that barring external issues, it should work fine for our use case.
<br><br>
**Decision:** Our final decision was to use PTC fusing for all power rails except two digital offboarded rails, which we would use e-fusing on for debugging purposes. Normally, the voltage drop across the shunt resistor in current-sensing OCP would affect analog readings, but would be negligible for digital signals. Therefore, we employed current-sensing OCP on two offboarded digital rails.
<br><br>
**ADC selection**
<br><br>
ADCs (analog-to-digital converters) take in an analog input and return a numerical value in binary to be processed by an MCU. On the sensor boards, the ADCs are responsible for handling the analog sensor output data and converting it to digital signals to be logged over CAN.
*insert screenshot of altium schematic and layout of ADCs from 2025-2026*
*explain background on our 2025-2026 ADC*
*explain goals for 2026-2027 season with the trade*
*insert trades (including two architectures of ADC and their differences) and conclusion here*
<br><br>
**Filtering and buffering**
Between the sensor data output and ADC input, we have voltage buffers and an RC filter to remove noise from the readings.
<br><br>
**Grounding**
For the 2025-2026 season, we ran separate analog and digital grounds on the sensor boards. Analog grounds are quiet, have low current, and therefore a low, uniform voltage. This is beneficial for the ADC, which measures differential voltage between its input pin and GND. Same GND reference between the sensors and ADC input yields greater accuracy.
<br><br>
Digital grounds are for the digital, switching components. Fast switching introduces significant noise and ground bounce, where the voltage of the ground plane increases above 0V (usually in the millivolts range). For digital signals, however, we don't care, because most components such as the MCU measure digital low vs. high at around 3.3V. Noise or ground bounce does not affect the readings, and so fluctuations in the signal, unlike in analog components, do not impact normal function.
<br><br>
*make list of pros and cons to this*
*explain decision to combine grounds in 2026-2027*
<br><br>
We presented on these trades in an intermediate design review (IDR); the presentation can be found in [intermediate-design-review.pdf](intermediate-design-review.pdf). I worked on the sensor powering OCP and ADC selection, while my teammates worked on the other trades.
<br><br>
## Switch to dual sensor board architecture
**Increased current draw**
<br><br>
**MCU pinout:**

