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
*insert screenshot of confluence trade study here*
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
**Switch to dual sensor board architecture**
