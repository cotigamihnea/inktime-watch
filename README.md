# InkTime Smartwatch - Proiect Hardware

Acesta este repository-ul oficial pentru dezvoltarea hardware a proiectului InkTime, un smartwatch bazat pe tehnologia E-Paper și procesorul nRF52840, optimizat pentru un consum redus de energie.

---

## 1. Diagramă Bloc

![Diagrama Bloc](Images/pic1.png)

---

## 2. Bill of Materials (BOM) Principale

Mai jos sunt listate componentele integrate (IC) principale folosite în designul acestui smartwatch. Fișierul BOM complet cu toate componentele pasive (rezistențe, condensatoare) se găsește în folderul `Manufacturing`.

| Componentă / Modul | Rol în Sistem | JLC Parts Link | Datasheet |
| :--- | :--- | :--- | :--- |
| **nRF52840-QIAA** | Microcontroller Principal (BLE 5.0) | [C209930](https://jlcpcb.com/partdetail/Nordic_Semiconductor-nRF52840_QIAAR/C209930) | [Datasheet nRF52840](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.1.pdf) |
| **BQ25180YBGR** | Li-Ion Battery Charger (PMIC) | [C3005391](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C3005391) | [Datasheet BQ25180](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| **MAX17048G+T10** | Fuel Gauge (Monitorizare Baterie) | [C137599](https://jlcpcb.com/partdetail/Maxim_Integrated-MAX17048GT10/C137599) | [Datasheet MAX17048](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| **BMA421** | IMU / Accelerometru (Pedometer) | [C456073](https://jlcpcb.com/partdetail/Bosch-BMA421/C456073) | [Datasheet BMA421](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma421-fl000.pdf) |
| **503480-2400** | Conector Display E-Paper (24 pini) | [C265215](https://jlcpcb.com/partdetail/Molex-5034802400/C265215) | [Datasheet Molex](https://www.molex.com/pdm_docs/sd/5034802400_sd.pdf) |

---

## 3. Descrierea Funcționalității Hardware

Proiectul InkTime este gândit ca un wearable ultra-low-power. Arhitectura hardware este împărțită în următoarele module principale:

* **Unitatea de Procesare și Comunicație (nRF52840):** Este "creierul" plăcii. Acesta conține un nucleu ARM Cortex-M4F și un modul radio Bluetooth Low Energy (BLE 5.0). Microcontroller-ul gestionează toate perifericele și permite sincronizarea datelor cu un smartphone.
* **Afișajul (E-Paper Display):** Conectat printr-o interfață **SPI**, display-ul E-paper a fost ales pentru consumul său zero de energie în starea de repaus (consumă curent doar la refresh-ul ecranului). Circuitul său include o sursă de tensiune dedicată, cu o bobină și diode, comandată de un tranzistor DMG2305UX, pentru a-i asigura tensiunile specifice de operare.
* **Managementul Energiei (Power Management):** * **BQ25180:** Se ocupă de încărcarea sigură a bateriei Li-Po de la mufa USB (5V), setând curenți mici de încărcare adecvați bateriilor de smartwatch.
    * **MAX17048 (Fuel Gauge):** Citește tensiunea bateriei și calculează procentul (State of Charge - SoC) folosind algoritmul ModelGauge. Ambele comunică cu nRF-ul prin magistrala **I2C**.
* **Senzori și Haptic:** Accelerometrul **BMA421** este folosit pentru detectarea mișcării (ex: ridicarea mâinii pentru a aprinde ecranul) și numărarea pașilor, comunicând tot prin **I2C**. Pentru feedback tactil s-a integrat un driver haptic.
* **Considerații de Consum:** Designul este centrat pe eficiență. nRF52840 va sta majoritatea timpului în *Deep Sleep* (câțiva microamperi), trezindu-se doar la întreruperi generate de BMA421 (mișcare) sau la intervale regulate pentru a actualiza ecranul prin SPI.

---

## 4. Pinout nRF52840 (Alocarea Pinilor)

Microcontroller-ul comunică cu restul modulelor folosind interfețe digitale standardizate. Magistrala I2C este partajată pentru a economisi pini.

| Periferic / Modul | Pini nRF52840 Alocați | Interfață | Motiv / Funcție |
| :--- | :--- | :--- | :--- |
| **Magistrala I2C (Comună)** | `P0.11` (SDA), `P0.12` (SCL) | I2C | Folosită pentru BQ25180 (Charger), MAX17048 (Baterie) și BMA421 (IMU). I2C permite legarea mai multor senzori pe doar 2 pini fizici, reducând complexitatea rutării. |
| **BMA421 (Senzor Mișcare)**| `P0.13` (INT1) | GPIO | Pin de întrerupere pentru a „trezi” procesorul din sleep când utilizatorul face o mișcare bruscă. |
| **Display E-Paper** | `P1.01` (MOSI), `P1.02` (SCK), `P1.03` (CS) | SPI | Interfață de mare viteză unidirecțională pentru a trimite pixelii către ecran rapid. |
| **Control E-Paper** | `P1.04` (DC), `P1.05` (RST), `P1.06` (BUSY)| GPIO | Pini de control pentru stările ecranului (Data/Command, Reset și monitorizarea stării de Busy în timpul refresh-ului). |
| **Alimentare E-Paper** | `P0.06` | GPIO | Controlează tranzistorul care taie complet alimentarea ecranului când nu este folosit, salvând energie. |

*(Notă: Pinii exacți au fost aleși pentru a facilita cel mai scurt drum de rutare (Placement Optimization) pe PCB).*

---

## 5. Design Log & Renders

**Jurnal de Design:**
Proiectarea PCB-ului a presupus mai multe provocări:
1.  **Placement:** Componentele au fost grupate în "cartiere" logice: RF-ul sus în jurul antenei pentru a minimiza interferențele, IC-urile de Power Management lângă conectorul bateriei/USB, iar componentele digitale centralizate în jurul procesorului nRF52840.
2.  **Routing:** Datorită densității mari de pe o placă cu factor de formă mic (Smartwatch), am utilizat un mix de rutare manuală pentru traseele sensibile și Autorouter pentru optimizarea conexiunilor logice, obținând o rată de conectare excelentă pentru un sistem cu doar 2 straturi.

**Randări PCB:**
![Top Render](Images/pic3.png)
![Bottom Render](Images/pic2.png)