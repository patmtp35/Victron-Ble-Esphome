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

🤝 Contributeurs & Ressources

ESPHOME Victron BLE – Fabian Schmidt
https://github.com/Fabian-Schmidt/esphome-victron_ble

ESPHOME Victron VE.Direct – KinDR007
https://github.com/KinDR007/VictronMPPT-ESPHOME

Merci à eux pour le travail de base sur lequel repose ce projet. 
