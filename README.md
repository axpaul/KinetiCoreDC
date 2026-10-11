# KinetiCoreDC

Carte de contrôle moteur basée sur un microcontrôleur **STM32G0** (STM32G070KBT6) conçue pour piloter des moteurs DC à l'aide de ponts en H intégrés **DRV8874** (Texas Instruments).

## Fonctionnalités principales

- **Microcontrôleur :** STM32G070KBT6 (ARM Cortex-M0+)
- **Driver moteur :** DRV8874 (pilotage moteur DC avec mesure de courant et protections intégrées)
- **Communication & Interfaces :**
  - Bus RS485 (transceiver THVD1400)
  - Bus I2C avec extenseur d'E/S (TCA9534A)
  - Entrées isolées / optocoupleurs (TLP2310)
- **Alimentation & Protections :**
  - Contrôleur de surtension / sous-tension et protection contre les inversions de polarité (LTC4365)
  - Régulateur LDO 3.3V (MCP1703)

## Structure du projet

- `KinetiCoreDC.kicad_pro` : Fichier de projet KiCad
- `KinetiCoreDC.kicad_sch` & `Interface.kicad_sch` : Schémas électroniques
- `KinetiCoreDC.kicad_pcb` : Routage du circuit imprimé
- `Datasheet/` : Documentation technique des composants clés
- `Library/` : Empreintes et modèles de composants

## Licence

Voir le fichier [LICENSE](LICENSE) pour plus de détails.
