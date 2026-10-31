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
<br?>br>
For the 2025-2026 season, I ran trade studies on potential OCP components such as PTC and automotive fusing, e-fusing, load-switch ICs, ORing controllers, and op-amps.
<br><br>
*insert screenshot of trade study here*
<br><br>
Ultimately, we decided on PTC fusing with an op-amp buffer as our protection circuit. The PTC fusing provides OCP while the op-amp provides voltage buffering. This is the OCP circuit for the 3V3D rail:
<br><br>
<img width="1917" height="375" alt="image" src="https://github.com/user-attachments/assets/111310ce-256a-4d1a-9b82-190f8ba20e47" />

