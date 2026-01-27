# Victron Unified ESP  
**BLE + VE.Direct redondant → Venus OS Ready (ESS via MQTT)**

2× SmartSolar MPPT (100/20 + 100/30) + SmartShunt 500A  
ESP32 unique – multi-sources – haute résilience

---

## ⚠️ Avertissement

Projet personnel avancé, conçu pour un système Victron réel.  
Ce dépôt **n’est pas un template universel**, mais une configuration ESPHome :

- orientée **résilience**
- compatible **Home Assistant existant**
- et désormais **compatible Venus OS / ESS via MQTT**

Les versions **V5.x et V6.0 coexistent** et sont **toutes fonctionnelles**.

---

## 📌 Description générale

Projet ESPHome permettant de lire **simultanément et en parallèle** les données Victron via :

- **Bluetooth Low Energy (BLE)**
- **VE.Direct (UART)**
- **MQTT (Venus OS / ESS – à partir de la V6.0)**

Objectifs :
- Redondance automatique BLE ↔ VE.Direct
- Continuité de service Home Assistant
- Préparation native à une intégration **Venus OS (ESS, DVCC)**

---

## 🧱 Versions du projet

### 🔵 V5.x — BLE + VE.Direct redondant (stable)
✔ Fonctionnelle  
✔ Utilisée en production  
✔ Orientée Home Assistant  

Fonctionnalités :
- Lecture BLE + VE.Direct simultanée
- Templates unifiés (priorité BLE, fallback VE.Direct)
- Compatibilité rétro Home Assistant (anciens capteurs conservés)
- Redondance automatique sans coupure

👉 **Version recommandée si tu n’utilises PAS Venus OS**

---

### 🟢 V6.0 — Venus OS Ready (ESS via MQTT)
✔ Fonctionnelle  
✔ Compatible **Venus OS / ESS / DVCC**  
✔ Toujours compatible Home Assistant  

**Nouveautés V6.0 :**
- Publication MQTT conforme à l’arborescence Victron GX
- SmartShunt ESP vu comme *Battery Monitor*
- MPPT ESP vus comme *Solar Chargers*
- Pilotage ESS :
  - Mode ESS
  - Limites de courant MPPT
  - Autorisation charge / décharge (DVCC)
  - Power setpoint batterie
- Aucune dépendance à dbus direct → **MQTT only**

👉 **Version recommandée si tu prévois d’installer Venus OS**

---

## 🔧 Matériel compatible

- Victron SmartSolar MPPT 100/20
- Victron SmartSolar MPPT 100/30
- Victron SmartShunt 500A
- ESP32 (BLE + ≥3 UART)
- Home Assistant + ESPHome
- *(optionnel)* Raspberry Pi + Venus OS

---

## ✨ Fonctionnalités détaillées

### ✔ Télémetrie BLE (rapide & redondante)

Données disponibles :
- Tension batterie
- Courant batterie
- Puissance batterie / PV
- Tension panneau
- Courant Load
- Température MPPT
- Production du jour
- SOC (SmartShunt)

👉 BLE = **rapide, sans fil, mais données partielles**

---

### ✔ Télémetrie VE.Direct (complète & fiable)

Données disponibles :
- Panel voltage / power
- Battery voltage / current
- Load current
- Yield today / yesterday / total
- Max power today / yesterday
- Charging mode / tracking mode
- Codes erreur
- Firmware, type, numéro de série
- Consumed Ah (SmartShunt)

👉 VE.Direct = **référence principale**

---

### ✔ Redondance automatique (BEST source)

Les capteurs critiques utilisent des templates intelligents :
- Priorité BLE si disponible
- Fallback VE.Direct automatique
- Calculs de secours (V × I)

➡️ Aucune coupure Home Assistant  
➡️ Dashboards et automatisations inchangés

---

### ✔ MQTT Victron GX (V6.0)

Publication conforme :
- `N/ESP/system/0/Dc/Battery/*`
- `N/ESP/solarcharger/{0,1}/*`

Résultat :
- Venus OS détecte automatiquement :
  - SmartShunt ESP
  - MPPT 100/20 ESP
  - MPPT 100/30 ESP

👉 **Même structure qu’un Cerbo GX**

---

## 🧠 Architecture générale

- BLE actif en permanence
- 3 UART dédiés (1 par appareil VE.Direct)
- Aucun partage d’UART (half-duplex VE.Direct)
- Capteurs VE.Direct majoritairement `internal`
- Entités exposées Home Assistant stables et unifiées

---

## 🔌 Câblage VE.Direct  
### (2 MPPT + 1 SmartShunt)

### Règle essentielle
❌ Impossible de partager un UART  
✔ 1 appareil Victron = 1 UART

| Appareil | RX ESP32 | TX ESP32 | UART |
|--------|---------|---------|------|
| MPPT 100/20 | GPIO19 | GPIO18 (NC) | UART |
| MPPT 100/30 | GPIO16 | GPIO17 (NC) | UART |
| SmartShunt 500A | GPIO22 | GPIO21 (NC) | UART |

- TX Victron → RX ESP32
- TX ESP32 non utilisé
- Masse commune obligatoire

---

## 🧩 Home Assistant & compatibilité

- Anciens capteurs `smartsolar1_esp_*` recréés via templates
- Aucun dashboard cassé
- BLE, VE.Direct et MQTT coexistent
- Bouton restart conservé

---

## 🧪 Philosophie du projet

- Résilience avant tout
- Pas de dépendance à une seule technologie
- BLE = continuité
- VE.Direct = précision
- MQTT = intégration système (ESS)
- ESPHome lisible, maintenable, évolutif

---

## 🤝 Ressources & crédits

- ESPHome Victron BLE – Fabian Schmidt  
  https://github.com/Fabian-Schmidt/esphome-victron_ble

- ESPHome Victron VE.Direct – KinDR007  
  https://github.com/KinDR007/VictronMPPT-ESPHOME

🙏 Merci à eux pour le travail fondamental sur lequel repose ce projet.
