# data-acquisition-boards
The **data acquisition** (or sensor) boards on the vehicle is responsible for powering and protecting sensors, filtering analog and digital telemetry data, and converting it to CAN to be logged. It is non-car-critical, meaning nothing on the board should be required for the vehicle to drive.
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
<img width="1609" height="930" alt="image" src="https://github.com/user-attachments/assets/cb0ed0fa-8d8d-415d-930e-e2b4ada6d3d1" />
<br><br>
I/O is offboarded in the diagram.
<br><br>
Power stage:
<br><br>
<img width="1200" height="618" alt="image" src="https://github.com/user-attachments/assets/a2ec39bf-a832-41f6-88ca-4a642f602978" />

