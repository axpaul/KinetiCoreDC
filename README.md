# KinetiCoreDC

![MCU](https://img.shields.io/badge/MCU-STM32G070CBT6-03234B?style=flat-square&logo=stmicroelectronics)
![Driver](https://img.shields.io/badge/Driver-DRV8874PWPR-CC0000?style=flat-square&logo=texasinstruments)
![EDA](https://img.shields.io/badge/KiCad-10.0.6-314CB0?style=flat-square&logo=kicad)
![License](https://img.shields.io/badge/License-CERN--OHL--S--2.0-blue?style=flat-square)

**KinetiCoreDC** est un contrôleur de moteur DC en boucle fermée permettant de transformer tout moteur à balais en un **servomoteur intelligent** (asservissement en position, vitesse et couple). Développé par **Paul Miailhe** pour **Berry Rocket**.

© 2025/2026 - **Paul Miailhe** (*Berry Rocket*).  
This material is licensed under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S)**. $\rightarrow$ [LICENSE](LICENSE)

---

## 1. Vue d'ensemble & Applications

Articulée autour d'un microcontrôleur **STM32G070CBT6** (ARM Cortex-M0+ @ $64\text{ MHz}$) et d'un pont en H intégré **DRV8874PWPR**, la carte intègre l'ensemble de l'électronique de puissance, de protection et de conditionnement nécessaire au pilotage d'un actionneur DC en environnement embarqué :

- **Servomoteur numérique multi-axes :** Asservissement PID (position / vitesse / couple) piloté et supervisé sur bus différentiel RS-485 (`THVD1400D`, jusqu'à $500\text{ kbps}$).
- **Actionnement sécurisé ou redondant :** Pilotage par consigne PWM ou impulsion RC ($1\text{–}2\text{ ms}$) via deux entrées opto-isolées (`TLP2310`) autorisant une voie de commande secondaire de secours.
- **Mécatronique & Robotique mobile :** Contrôle de moteurs DC avec retour d'encodeur absolu (`AS5047P`, `AS5600`) ou incrémental Hall, prise d'origine (*homing*) sur butées mécaniques et limitation d'effort par mesure de courant (`IPROPI`).

---

## 2. Caractéristiques électriques

L'étage d'entrée est dimensionné pour une alimentation sur batterie **LiPo 2S ou 3S** (ou alimentation DC régulée). Un contrôleur **LTC4365TS8** associé à deux MOSFETs N-Channel tête-bêche (**SiS412DN**) assure la coupure matérielle en cas de sous-tension, surtension ou inversion de polarité, tandis que le driver **DRV8874** régule activement le courant moteur cycle par cycle.

| Paramètre | Symbole | Min | Typ | Max | Unité | Conditions / Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Tension d'entrée** | $V_{\text{IN}}$ | $5{,}90$ | $7{,}4\text{–}11{,}1$ | **$14{,}75$** | $\text{V}$ | Compatible LiPo 2S ($8{,}4\text{ V}$) et 3S ($12{,}6\text{ V}$) |
| **Tenue inversion polarité** | $V_{\text{REV}}$ | **$-40$** | — | — | $\text{V}$ | Blocage par N-MOSFETs tête-bêche |
| **Seuil sous-tension (coupure)** | $V_{\text{UVLO}}$ | — | $5{,}90$ | — | $\text{V}$ | Enclenchement à $6{,}20\text{ V}$ (config. défaut) |
| **Seuil surtension (coupure)** | $V_{\text{OVLO}}$ | — | **$14{,}75$** | — | $\text{V}$ | Réarmement à $14{,}01\text{ V}$ (LDO max $16\text{ V}$) |
| **Courant moteur régulé** | $I_{\text{TRIP}}$ | — | **$2{,}6$** | **$2{,}6$** | $\text{A}$ | $V_{\text{REF}} = 2{,}50\text{ V}$, $R_{\text{IPROPI}} = 2{,}49\text{ k}\Omega$ |
| **Courant moteur crête** | $I_{\text{PEAK}}$ | — | — | **$6{,}0$** | $\text{A}$ | Limite silicium `DRV8874` (`XT30` : $15\text{ A}$ cont.) |
| **Puissance d'entrée** | $P_{\text{IN}}$ | — | $32{,}8$ | $38{,}4$ | $\text{W}$ | À $I_{\text{TRIP}} = 2{,}6\text{ A}$ (jusqu'à $88{,}5\text{ W}$ crête à $6\text{ A}$) |
| **Rail logique** | $V_{\text{3V3}}$ | $3{,}28$ | $3{,}30$ | $3{,}32$ | $\text{V}$ | Précision $\pm 0{,}4\text{ \%}$ (`MCP1703A`) |
| **Courant disponible $+3{,}3\text{ V}$** | $I_{\text{OUT(3V3)}}$ | — | — | **$250$** | $\text{mA}$ | Total carte ($\approx 50\text{ mA}$) + capteurs externes |
| **Courant de veille** | $I_{\text{SHDN}}$ | — | — | **$< 10$** | $\mu\text{A}$ | Broche `SHDN_EXT` tirée à `GND` |

> **Ajustement des seuils :** La fenêtre de protection UVLO/OVLO peut être adaptée à un profil batterie spécifique (LiPo 2S dédié $6{,}45\text{–}10{,}84\text{ V}$, LiPo 3S dédié $9{,}15\text{–}14{,}28\text{ V}$ ou pile 9V) via le pont diviseur du `LTC4365` sur la feuille `Power.kicad_sch`. De même, le seuil de courant $I_{\text{TRIP}}$ est ajustable jusqu'à $6\text{ A}$ via `R_IPROPI`.

---

## 3. Interfaces & Capteurs supportés

Toutes les entrées/sorties externes sont équipées de protections dédiées contre les décharges électrostatiques (ESD), les inversions de câblage et le bruit de découpage moteur ($32\text{ kHz}$). Les signaux mécaniques (encodeur et butées) sont filtrés matériellement puis remis en forme par des triggers de Schmitt (**SN74LVC2G14DBV**).

| Connecteur | Interface | Signal / Protocole | Conditionnement & Protection | Compatibilité |
| :--- | :--- | :--- | :--- | :--- |
| **`J2` / `J13`** | `XT30PW-M` (2 pts) | Puissance $V_{\text{IN}}$ / Moteur | TVS `SMBJ30CA`, N-MOSFETs, LEDs sens/PWM | LiPo 2S–3S, moteur DC à balais |
| **`J8`** | `JST-GH` (3 pts) | RS-485 Half-Duplex ($\le 500\text{ kbps}$) | `THVD1400D`, terminaison $120\text{ }\Omega$, TVS `SM712` | Bus multi-axes, télémétrie |
| **`J4` / `J5`** | `JST-GH` (2 pts) | $2\times$ PWM / GPIO isolés ($3{,}3\text{–}5\text{ V}$) | Optocoupleurs `TLP2310`, clamp inverse `SMF6.0A` | Consigne PWM/RC, voie redondante |
| **`J11`** | `JST-GH` (6 pts) | Bus hybride `SPI2` ou `I2C1` | TVS 4 voies `ESD224DQA` ($0{,}5\text{ pF}$), pull-ups `DNP` | **AS5047P**, **AS5048A**, **MA730**, **AS5600** |
| **`J3`** | `JST-GH` (4 pts) | Encodeur quadrature A/B ($3{,}3\text{–}5\text{ V}$) | `USBLC6-2SC6` + RC ($f_c \approx 160\text{ kHz}$) + Schmitt | Encodeurs incrémentaux à effet Hall |
| **`J6` / `J7`** | `JST-GH` (2 pts) | $2\times$ Fins de course (contact sec) | Pull-up $2{,}2\text{ k}\Omega$, RC ($f_c \approx 160\text{ Hz}$) + Schmitt | Butées mécaniques Min / Max |
| **`J10`** | `JST-GH` (3 pts) | Activation carte (`SHDN_EXT`) | Commande directe broche `SHDN` du `LTC4365` | Interrupteur marche / veille |
| **`J12`** | `JST-GH` (2 pts) | Sortie défaut alim (`NFAULT_EXT`) | Drain ouvert actif bas (`LTC4365`) | Supervision système hôte |
| **`J15`** | `TC2070-NL` (14 pts) | SWD + NRST + UART VCP (`USART3`) | Empreinte sans composants (STDC14) | ST-LINK/V3 (flash & debug) |

Pour les capteurs de position absolue sur **`J11`**, le bus fonctionne par défaut en **SPI haute vitesse** (recommandé en environnement bruité pour les **AS5047P** / **MA730**). Il peut être basculé en **I²C** pour un **AS5600** en soudant les deux résistances de pull-up de $4{,}7\text{ k}\Omega$ prévues sur la feuille `Connector.kicad_sch`.

---

## 4. Brochage MCU (`STM32G070CBT6` — LQFP48)

L'affectation des broches du microcontrôleur est optimisée par zone physique afin de regrouper l'ensemble des signaux de commande du pont en H sur le flanc droit du boîtier et d'isoler les mesures analogiques (`IPROPI` et `ADC_VM`).

| Bloc fonctionnel | Signal | Broche | Périphérique | Fonction |
| :--- | :--- | :---: | :--- | :--- |
| **Driver Moteur** | `EN/IN1_MCU` | `PA8` (28) | `TIM1_CH1` | PWM principale moteur ($32\text{ kHz}$) |
| | `PH/IN2_MCU` | `PA9` (29) | `TIM1_CH2` / GPIO | Direction (`PH`) ou PWM secondaire |
| | `NSLEEP_MCU` | `PC6` (30) | `GPIO_Output` | Activation / Veille du pont en H |
| | `PMODE_MCU` | `PC7` (31) | `GPIO_Output` | Mode `PH/EN` (`0`) ou `PWM` (`1`) |
| | `NFAULT_MCU` | `PA10` (32) | `GPIO_EXTI10` | Interruption défaut driver (front descendant) |
| | `IPROPI_MCU` | `PB2` (21) | `ADC1_IN10` | Lecture courant moteur ($1{,}13\text{ V/A}$, filtre RC) |
| **Supervision** | `ADC_VM` | `PA0` (11) | `ADC1_IN0` | Mesure tension d'entrée $V_M$ |
| **RS-485** | `RS485_DE` / `TX` / `RX` | `PA1`, `PA2`, `PA3` | `USART2` (+ DMA) | Bus différentiel avec contrôle DE matériel |
| **Encodeur** | `TIM_ENC_A` / `B` | `PB4` (43), `PB5` (44) | `TIM3_CH1/CH2` | Décodage quadrature matériel X4 |
| **Optocoupleurs** | `OPTO1_MCU` / `OPTO2_MCU` | `PB14` (26), `PB15` (27) | `TIM15_CH1/CH2` | Input Capture PWM / Consigne redondante |
| **Fins de course** | `TRIG1_MCU` / `TRIG2_MCU` | `PD2` (40), `PD3` (41) | `GPIO_EXTI2/EXTI3` | Interruptions butées (front montant) |
| **Bus Capteur** | `MISO`/`MOSI`/`SCK`/`NSS` | `PB6`..`PB9` (45–48) | `SPI2` (ou `I2C1`) | Encodeur magnétique absolu |
| **IHM** | `RED` / `GREEN` / `BLUE` | `PA6`, `PA7`, `PB0` | `GPIO_Output` | LED RGB 8 couleurs (anode commune, actif bas) |
| | `BUZZER_MCU` | `PA4` (15) | `TIM14_CH1` | Signal piézo PWM $4\text{ kHz}$ |
| **Debug** | `SWDIO` / `SWCLK` | `PA13` (35), `PA14` (36) | `SYS_SWD` | Programmation et débogage |
| | `VPC_TX` / `VPC_RX` | `PB10` (22), `PB11` (23) | `USART3_TX/RX` | Console série virtuelle de diagnostic |

---

## 5. Architecture KiCad & Structure du dépôt

Le projet est conçu sous **KiCad 10.0.6** autour d'une feuille racine et de trois sous-feuilles hiérarchiques, accompagnées du fichier de configuration **STM32CubeMX** (`KinetiCoreDC.ioc`) :

```text
KinetiCoreDC/
├── KinetiCoreDC.kicad_pro       # Projet KiCad 10.0.6
├── KinetiCoreDC.kicad_sch       # Feuille 1/4 : MCU STM32G070CBT6 & hiérarchie
├── Interface.kicad_sch          # Feuille 2/4 : RS-485, optocoupleurs, encodeur, butées
├── Power.kicad_sch              # Feuille 3/4 : LTC4365, LDO 3.3V, DRV8874, RGB, buzzer
├── Connector.kicad_sch          # Feuille 4/4 : XT30, JST-GH, Tag-Connect, ESD224DQA
├── KinetiCoreDC.kicad_pcb       # Routage PCB
├── KinetiCoreDC.ioc             # Configuration matérielle STM32CubeMX
├── Datasheet/                   # Fiches techniques des composants
├── lib/                         # Symboles, empreintes et modèles 3D locaux
└── LICENSE                      # Licence CERN-OHL-S-2.0