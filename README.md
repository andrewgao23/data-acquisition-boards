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
<br>br>
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
*insert screenshot of confluence trade study here*
<br><br>
Many components were seriously considered for the final design. ORing controllers were an initial choice, but ultimately discounted as they were relatively complex to implement, required external components, and were not designed specifically for the function that we wanted it to perform (only as a side benefit).
<br><br>
Load-switch ICs were a promising choice; initially discounted due to a low voltage input range, upon verification with sensor voltage requirements we deemed it sufficient for our use case. However, these were new components not used elsewhere on the car, adding complexity and risk to implementation, and we found components better suited for our use case later on.
<br><br>
The final decision was between current-sensing e-fusing, automotive fusing, and PTC fusing.
*explain pros and cons for all three options and which one we decided*
<br><br>
**ADC selection**
<br><br>
How ADCs work and the two main architectures:
<br><br>
**Filtering**
<br><br>
**Grounding**
<br><br>
## Switch to dual sensor board architecture
<br><br>
**MCU pinout:**

