# 🔆 ESPHome Victron Unified Venus OS

[![ESPHome](https://img.shields.io/badge/ESPHome-2024.x-blue?logo=esphome)](https://esphome.io)
[![Venus OS](https://img.shields.io/badge/Venus%20OS-Ready-green)](https://github.com/victronenergy/venus)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Interface ESP32 unifiée pour intégrer des équipements **Victron Energy** dans **Venus OS** et **Home Assistant** via BLE + VE.Direct + MQTT.

---

## 📋 Fonctionnalités

- ✅ **Double source de données** par appareil : BLE (prioritaire) + VE.Direct (fallback)
- ✅ **2 MPPT SmartSolar 100/20** supportés simultanément
- ✅ **1 SmartShunt 500A** (courant, tension, SoC, Ah consommés)
- ✅ **Venus OS ready** via MQTT (compatible `dbus-mqtt-solar-charger` et `dbus-mqtt-devices`)
- ✅ **Home Assistant** via API ESPHome native
- ✅ Historique journalier complet (Yield, MaxPower, MaxPvVoltage, timers Bulk/Abs/Float)
- ✅ Toutes les valeurs configurables via les `substitutions`

---

## 🔧 Matériel requis

| Composant | Modèle testé |
|---|---|
| Microcontrôleur | ESP32 (esp32dev) |
| Chargeur solaire #1 | Victron SmartSolar MPPT 100/20 |
| Chargeur solaire #2 | Victron SmartSolar MPPT 100/20 |
| Moniteur batterie | Victron SmartShunt 500A |
| Passerelle domotique | Venus OS (Cerbo GX ou Raspberry Pi) |

---

## 📦 Dépendances ESPHome

```yaml
external_components:
  # BLE Victron
  - source: github://Fabian-Schmidt/esphome-victron_ble
  # VE.Direct Victron
  - source: github://KinDR007/VictronMPPT-ESPHOME@main
```

---

## 🌐 Dépendances Venus OS

Deux scripts à installer sur votre Venus OS :

### 1. dbus-mqtt-solar-charger (mr-manuel)
> https://github.com/mr-manuel/venus-os_dbus-mqtt-solar-charger

Un fichier de config par MPPT dans `/data/etc/dbus-mqtt-solar-charger/` :

```ini
[DEFAULT]
logging = WARNING
device_name = SmartSolar MPPT 100/20
device_instance = 20
timeout = 30
history_days = 14

[MQTT]
broker_address = 127.0.0.1
broker_port = 1883
tls_enabled = 0
username = votre_user
password = votre_password
topic = venus/solarcharger/mppt_100_20
```

> ⚠️ `history_days` doit être exactement **14**, sinon le script plante.

### 2. dbus-mqtt-devices (freakent)
> https://github.com/freakent/dbus-mqtt-devices/tree/v0.9.0dev

Utilisé pour publier le SmartShunt (batterie) dans Venus OS via le topic `device/victron_unified/`.

---

## 🗂️ Fichiers du projet

```
.
├── victron_unified_venus.yaml   # fichier principal ESPHome
├── .common.yaml                 # config commune (wifi, ota, api, logger...)
├── secrets.yaml                 # vos secrets (non versionné)
└── README.md
```

---

## ⚙️ Configuration — secrets.yaml

```yaml
# Réseau
ip_victron_unified: "192.168.1.xxx"
ip_gateway:         "192.168.1.1"
ip_dns:             "8.8.8.8"

# OTA / API
ota_pswd:           "votre_mot_de_passe"
smartsolar2key:     "votre_cle_api_esphome"

# MQTT Venus OS
mqtt_venusos_host:  "192.168.1.xxx"   # IP de votre Venus OS
mqtt_venusos_user:  "mqtt_user"
mqtt_venusos_pswd:  "mqtt_password"
venus_portal_id:    "xxxxxxxxxxxx"    # ID portail VRM (12 caractères)

# BLE MPPT 100/20 #1
mppt100_20_mac:     "XX:XX:XX:XX:XX:XX"
mppt100_20_key:     "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# BLE MPPT 100/20 #2
mppt100_20_2mac:    "XX:XX:XX:XX:XX:XX"
mppt100_20_2key:    "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# BLE SmartShunt 500A
smartshunt500a_mac: "XX:XX:XX:XX:XX:XX"
smartshunt500a_key: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

### Obtenir les clés BLE Victron
Dans l'app **VictorConnect** : Appareil → `...` → **Product info** → **Encryption data**

### Obtenir le Portal ID Venus OS
Dans Venus OS : **Settings** → **VRM online portal** → **VRM Portal ID**

---

## ⚙️ Configuration — substitutions principales

Toutes les options importantes sont dans la section `substitutions` en haut du fichier YAML :

```yaml
substitutions:
  # Identité de l'appareil
  name:           "victron-unifiedvenus-esp"
  friendly_name:  "Victron Unified"

  # Noms des appareils (préfixe des sensors dans HA)
  mppt1_name:     "smartsolar1"
  mppt2_name:     "smartsolar2"
  shunt_name:     "smartshunt500a"

  # Topics Venus OS (doivent correspondre aux configs mr-manuel)
  mppt1_venus_topic: "venus/solarcharger/mppt_100_20"
  mppt2_venus_topic: "venus/solarcharger/mppt_100_20_2"

  # Noms des services dbus-mqtt-devices
  mppt1_service_name:  "solarcharger1"
  mppt2_service_name:  "solarcharger2"
  shunt_service_name:  "battery1"

  # Capacité batterie installée (Ah)
  battery_capacity_ah: "150"

  # Pins VE.Direct UART
  mppt1_uart_rx_pin: GPIO19   # TX du MPPT1
  mppt2_uart_rx_pin: GPIO16   # TX du MPPT2
  shunt_uart_rx_pin: GPIO22   # TX du SmartShunt

  # Intervalles de publication
  interval_fast:   "5s"    # puissance, courant
  interval_medium: "10s"   # tension
  interval_slow:   "60s"   # énergie, yield

  # Taille JSON (augmenter si JSON overflow dans les logs)
  json_doc_size: "8192"
```

---

## 🔌 Câblage VE.Direct

Le protocole VE.Direct est **unidirectionnel** en lecture : seul le **RX** de l'ESP32 est connecté au **TX** de l'équipement Victron.

```
Victron VE.Direct port        ESP32
─────────────────────        ──────
Pin 1 : GND          ──────► GND
Pin 2 : Vcc (5V)     ──────► (non connecté, alimentez l'ESP séparément)
Pin 3 : TX           ──────► GPIO19 (MPPT1 RX)
Pin 4 : RX           ──────► (non connecté)
```

> ⚠️ Le niveau logique VE.Direct est en **5V**. Utilisez un diviseur de tension ou un level-shifter pour protéger l'ESP32 (3.3V).

---

## 📡 Architecture des données

```
┌─────────────────────────────────────────────────┐
│                    ESP32                         │
│                                                  │
│  BLE ◄──── MPPT1 ────► VE.Direct (UART1)        │
│  BLE ◄──── MPPT2 ────► VE.Direct (UART2)        │
│  BLE ◄──── Shunt ────► VE.Direct (UART3)        │
│                                                  │
│  Priorité : BLE en premier, VE.Direct en fallback│
└────────────────────┬────────────────────────────┘
                     │ MQTT
          ┌──────────┴──────────┐
          │                     │
    ┌─────▼──────┐        ┌─────▼──────┐
    │  Venus OS  │        │    Home    │
    │  (GX MQTT) │        │ Assistant  │
    └────────────┘        └────────────┘
```

---

## 🏠 Sensors exposés dans Home Assistant

### Par MPPT (×2)
| Sensor | Unité | Description |
|---|---|---|
| `*_esp_panel_voltage` | V | Tension panneau PV |
| `*_esp_panel_power` | W | Puissance PV |
| `*_esp_panel_current` | A | Courant sortie MPPT |
| `*_esp_battery_voltage` | V | Tension batterie |
| `*_esp_load_current` | A | Courant charge |
| `*_esp_yield_today` | kWh | Production du jour |
| `*_esp_yield_yesterday` | kWh | Production hier |
| `*_esp_yield_total` | kWh | Production totale |
| `*_esp_max_power_today` | W | Puissance max du jour |
| `*_esp_charging_mode` | — | Mode de charge (Bulk/Abs/Float) |

### SmartShunt
| Sensor | Unité | Description |
|---|---|---|
| `smartshunt500a_battery_voltage` | V | Tension batterie |
| `smartshunt500a_battery_current` | A | Courant (+ charge / - décharge) |
| `smartshunt500a_battery_power` | W | Puissance batterie |
| `smartshunt500a_soc` | % | État de charge |
| `smartshunt500a_consumed_ah` | Ah | Ampères-heures consommés |
| `smartshunt500a_vd_time_to_go` | min | Temps restant estimé |

---

## 🌍 Données envoyées à Venus OS

### MPPT (via dbus-mqtt-solar-charger)
- Tension et puissance PV
- Tension et courant batterie
- État de charge (Bulk / Absorption / Float)
- Historique J et J-1 : Yield, MaxPower, MaxPvVoltage
- Timers : TimeInBulk, TimeInAbsorption, TimeInFloat
- MaxBatteryCurrent, Min/MaxBatteryVoltage
- Production totale (lifetime)

### Batterie (via dbus-mqtt-devices)
- Tension, courant, puissance DC
- SoC, ConsumedAh, InstalledCapacity
- TimeToGo
- Min/MaxVoltage journaliers

---

## 🔍 Dépannage

### `[E][json:071]: JSON document overflow`
Augmentez `json_doc_size` dans les substitutions :
```yaml
json_doc_size: "12288"
```

### Venus OS n'affiche pas les MPPT
Vérifiez que `dbus-mqtt-solar-charger` tourne sur Venus OS :
```bash
svstat /service/dbus-mqtt-solar-charger.ttyS4
```
Et que le `topic` dans sa config correspond à `mppt1_venus_topic` dans le YAML.

### BLE ne reçoit rien
- Vérifiez MAC et clé de chiffrement dans `secrets.yaml`
- L'ESP32 doit être à moins de ~10m des équipements Victron
- Assurez-vous que le BLE est actif sur l'équipement Victron (VictorConnect → paramètres)

### `Last transmission too long ago` (VE.Direct)
Warning normal si le câble VE.Direct n'est pas branché. Vérifiez le câblage et le `rx_pin` correct dans les substitutions.

---

## 📝 Changelog

| Version | Description |
|---|---|
| **6.4** | Refactoring complet en substitutions — publication publique |
| 6.3 | Amélioration remontée historique MPPT |
| 6.2 | GX ESS amélioration |
| 6.1 | Amélioration MQTT Venus OS |
| 6.0 | Ajout Venus OS Ready via MQTT |
| 5.0 | Unified BLE + VE.Direct Redundant |

---

## 📄 License

MIT — libre d'utilisation, modification et redistribution avec attribution.

---

## 🙏 Remerciements

- [Fabian-Schmidt](https://github.com/Fabian-Schmidt/esphome-victron_ble) — composant BLE Victron pour ESPHome
- [KinDR007](https://github.com/KinDR007/VictronMPPT-ESPHOME) — composant VE.Direct pour ESPHome
- [mr-manuel](https://github.com/mr-manuel/venus-os_dbus-mqtt-solar-charger) — bridge MQTT → dbus Venus OS (solar charger)
- [freakent](https://github.com/freakent/dbus-mqtt-devices) — bridge MQTT → dbus Venus OS (devices)
