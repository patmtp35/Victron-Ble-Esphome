# Victron Unified ESP – SmartSolar BLE + VE.Direct (Fallback Auto)

Projet ESPHome permettant de lire simultanément les données de contrôleurs de charge **Victron SmartSolar MPPT** via :

- **Bluetooth Low Energy (BLE)**  
- **VE.Direct (UART)**  
- **Fallback automatique VE.Direct → BLE** si l’une des deux sources tombe  
- **Normalisation des noms d’entités Home Assistant**  
- **Templates BEST** pour regrouper automatiquement la meilleure donnée  
- **Dashboard Lovelace complet**

Attention en BLE on a pas toutes les remontées comme en VE-direct , ceci etant on a le minimum vital pour assuré la continuité.

Compatible avec :

- MPPT **100/20**
- MPPT **100/30**
- ESP32 (2× UART + BLE)
- Home Assistant + ESPHome

---

## ✨ Fonctionnalités principales

### ✔ Télémetrie complète Victron BLE  
Récupération en direct via BLE :
- Tension batterie
- Courant batterie
- Puissance PV
- Tension PV
- Courant Load
- Température
- Yield today

### ✔ Télémetrie complète VE.Direct  
Lecture fiabilisée par UART :
- Panel voltage/power
- Battery voltage/current
- Load current
- Yield total
- Yield yesterday/today
- Max power today/yesterday
- Tracking mode
- Error code

### ✔ Fallback automatique (BEST)  
Chaque donnée critique possède :

➡ **BEST = VE.Direct priorité, BLE en secours**

#### connexions 

câblage DOUBLE VE.Direct pour connecter deux MPPT Victron (100/20 + 100/30) sur un seul ESP32, de manière propre, sûre et 100 % fonctionnelle pour ESPHome.

🎯 Objectif :

MPPT 100/20 → UART2

MPPT 100/30 → UART1

BLE toujours actif en parallèle

Fallback BEST qui utilisera VE.Direct → BLE

🟦 1. Schéma général 

🔥 IMPORTANT : chaque MPPT doit avoir son propre UART.

On ne peut PAS partager un même UART entre 2 VE.Direct (protocol simple half-duplex).

Donc :

MPPT	ESP32 Pin	UART
SmartSolar 100/20	GPIO 16 (RX2)	UART2
SmartSolar 100/30	GPIO 4 (RX1)	UART1
Masse commune	GND	-
🟦 2. Schéma visuel détaillé
               +---------------------------+
               |         ESP32             |
               |                           |
               |   UART2          UART1    |
               |   ------         ------   |
               |  RX2 = 16       RX1 = 4   |
               |  TX2 = 17 (NC)  TX1 = 5(NC)|
               |                           |
               +---------------------------+
                     |                |
                     |                |
      +--------------+                +--------------+
      |                                              |
      |                                              |
+-----------+                                  +-----------+
| MPPT      |                                  | MPPT      |
| 100/20    |                                  | 100/30    |
|           |                                  |           |
| VE.Direct |                                  | VE.Direct |
|   TX ---- +---------------------------------> RX1 (GPIO4)  
|   GND ---+-----> GND                          GND ----> GND
|           |                                  |           |
| (RX NC)   |                                  | (RX NC)   |
+-----------+                                  +-----------+



✔ TX du MPPT va toujours vers RX du ESP32
✔ Les pins TX du ESP ne sont pas utilisés
✔ Une seule masse commune pour tout le montage
✔ Distances courtes, câble torsadé recommandé

🟦 3. Pins conseillés pour ESP32 (DevKit v1)
UART	RX Pin	TX Pin	Utilisation
UART2	GPIO 16	GPIO 17	MPPT 100/20
UART1	GPIO 4	GPIO 5	MPPT 100/30

Les pins 4 et 5 sont parfaitement compatibles avec UART1 et disponibles sans conflit.

🤝 Contributeurs & Ressources

ESPHOME Victron BLE – Fabian Schmidt
https://github.com/Fabian-Schmidt/esphome-victron_ble

ESPHOME Victron VE.Direct – KinDR007
https://github.com/KinDR007/VictronMPPT-ESPHOME

Merci à eux pour le travail de base sur lequel repose ce projet. 
