# 🔆 ESPHome Victron Unified Venus OS

[![ESPHome](https://img.shields.io/badge/ESPHome-2024.x-blue?logo=esphome)](https://esphome.io)
[![Venus OS](https://img.shields.io/badge/Venus%20OS-Ready-green)](https://github.com/victronenergy/venus)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Interface ESP32 unifiée pour intégrer des équipements **Victron Energy** dans **Venus OS** et **Home Assistant** via BLE + VE.Direct + MQTT.

---

## 🔀 Quelle version choisir ?

| Fichier | Pour quelle installation |
|---|---|
| `victron_unified_venus_v6.5.yaml` | L'ESP lit **2 MPPT et un SmartShunt**. Le SmartShunt est publié dans Venus OS comme batterie virtuelle (`dbus-mqtt-devices`). |
| `victron_unified_venus_v6.6.yaml` | L'ESP lit **les 2 MPPT seulement**. La batterie vient directement du GX (par exemple un BMS en CAN). |

La **v6.6 est une évolution adaptée à ma configuration** (batterie avec BMS relié au GX). Elle ne remplace pas la v6.5, conservée pour les installations comme la précédente. La v6.5 n'est plus testée sur mon installation.

> ⚠️ En passant de la v6.5 à la v6.6, la batterie virtuelle disparaît : choisissez **avant** un autre moniteur de batterie sur le GX, et vérifiez vos automatisations, écrans et tableaux de bord qui lisent le SmartShunt.
> La v6.6 ne contient pas de message de désinscription de l'ancienne batterie virtuelle `battery1` : vérifiez qu'elle a disparu du GX.

---

## 📋 Fonctionnalités

- ✅ **Double source de données** par appareil : BLE + VE.Direct (priorité selon la mesure)
- ✅ **2 MPPT SmartSolar 100/20** supportés simultanément
- ✅ **1 SmartShunt 500A** (courant, tension, SoC, Ah consommés) — *v6.5 uniquement*
- ✅ **Venus OS ready** via MQTT (`dbus-mqtt-solar-charger` ; `dbus-mqtt-devices` en v6.5 uniquement)
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
| Moniteur batterie (v6.5) | Victron SmartShunt 500A |
| Passerelle domotique | Venus OS (Cerbo GX ou Raspberry Pi) |

---

## 📦 Dépendances ESPHome

La **v6.6 fige** les composants externes sur les commits testés (ESPHome 2026.11.1) :

```yaml
external_components_source_ble:      github://Kendzione/esphome-victron_ble@b9c6358b72320fd89d54f4d52614ec6d53fe6849
external_components_source_vedirect: github://KinDR007/VictronMPPT-ESPHOME@f5e9b989f76bee745c2581ee75543673c2ea5487
```

La **v6.5** suit les dernières versions (elle peut donc cesser de compiler après une évolution d'un composant) :

```yaml
external_components:
  # BLE Victron (fork Kendzione, compatible ESPHome >= 2026.11)
  - source: github://Kendzione/esphome-victron_ble
  # VE.Direct Victron
  - source: github://KinDR007/VictronMPPT-ESPHOME@main
```

> ⚠️ **ESPHome ≥ 2026.11** : le composant d'origine
> [Fabian-Schmidt/esphome-victron_ble](https://github.com/Fabian-Schmidt/esphome-victron_ble)
> ne compile plus (`'str_snprintf' was not declared in this scope`), car ESPHome 2026.11.0
> n'exporte plus `alloc_helpers.h` depuis `helpers.h`. Ce dépôt utilise donc le fork
> [Kendzione/esphome-victron_ble](https://github.com/Kendzione/esphome-victron_ble).
> Pour plus de stabilité, vous pouvez figer un commit : `github://Kendzione/esphome-victron_ble@<hash>`.
> Avec une version d'ESPHome antérieure à 2026.11, le composant d'origine fonctionne aussi.

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

### 2. dbus-mqtt-devices (freakent) — v6.5 uniquement
> https://github.com/freakent/dbus-mqtt-devices/tree/v0.9.0dev

Utilisé pour publier le SmartShunt (batterie) dans Venus OS via le topic `device/victron_unified/`.

---

## 🗂️ Fichiers du projet

```
.
├── victron_unified_venus_v6.5.yaml  # MPPT + SmartShunt
├── victron_unified_venus_v6.6.yaml  # MPPT seuls
├── .common.yaml                 # config commune (wifi, ota, api, logger...) : non fournie, à créer
├── secrets.yaml                 # vos secrets (non versionné)
└── README.md
```

---

## ⚙️ Configuration — secrets.yaml

> **v6.6** : `venus_portal_id` et les deux secrets `smartshunt500a_*` ne sont plus nécessaires.

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

## 🔵 v6.6 : associer chaque MPPT à ses identifiants BLE

`mppt1_ble_mac` / `mppt1_ble_key` doivent désigner **le même MPPT** que celui câblé sur `mppt1_uart_rx_pin` (idem pour le MPPT 2). Dans le fichier v6.6, `mppt1` utilise les secrets `mppt100_20_2*` et `mppt2` les secrets `mppt100_20_*` : c'est propre à mon installation, **adaptez-le à la vôtre**.

Vérification : pour un même numéro, la production du jour et la puissance doivent concorder entre les capteurs `*_ble_*` et `*_vd_*`.

---

## ⚙️ Configuration — substitutions principales (exemple v6.5)

> En v6.6, les substitutions `shunt_*`, `*_service_name` et `battery_capacity_ah` n'existent plus.

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

## 📡 Architecture des données (v6.5)

> **v6.6** : pas de SmartShunt. Pour la publication des MPPT vers le GX, le VE.Direct est utilisé en premier. La tension batterie a un secours BLE du même MPPT, comme la puissance et la tension PV si le VE.Direct n'a fourni aucune valeur. Les capteurs `*_esp_battery_voltage` de Home Assistant utilisent le BLE en premier.

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

### SmartShunt (v6.5)
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

### Batterie (via dbus-mqtt-devices, v6.5)
- Tension, courant, puissance DC
- SoC, ConsumedAh, InstalledCapacity
- TimeToGo
- Min/MaxVoltage journaliers

---

## ⚠️ Limites connues (v6.6)

- La bascule sur le BLE en cas de perte du VE.Direct **n'a pas été testée**. Après une perte de liaison en cours de fonctionnement, l'ESP peut conserver et republier la dernière valeur. Le courant batterie publié n'a pas de secours BLE.
- Après un redémarrage de l'ESP, certaines données d'historique (dont celles de la veille) repartent à zéro côté GX.

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

### Erreur de compilation `'str_snprintf' was not declared in this scope`
Le composant BLE d'origine n'est pas compatible avec ESPHome ≥ 2026.11. `validate` passe, mais la compilation échoue dans `victron_ble.h`.
Utilisez le fork indiqué dans la section *Dépendances ESPHome* (`github://Kendzione/esphome-victron_ble`).

### `Last transmission too long ago` (VE.Direct)
Warning normal si le câble VE.Direct n'est pas branché. Vérifiez le câblage et le `rx_pin` correct dans les substitutions.

---

## 📝 Changelog

| Version | Description |
|---|---|
| 6.6 | MPPT seuls : retrait du SmartShunt, de `dbus-mqtt-devices` et du keepalive ; composants externes figés ; tension de secours et min/max par MPPT ; publication seulement si une tension MPPT est disponible |
| 6.5.1 | Composant BLE : fork Kendzione (compilation avec ESPHome ≥ 2026.11) |
| 6.5 | VE.Direct prioritaire sur données PV (fix croisement BLE/VE.Direct) |
| 6.4 | Refactoring complet en substitutions — publication publique |
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

- [Fabian-Schmidt](https://github.com/Fabian-Schmidt/esphome-victron_ble) — composant BLE Victron pour ESPHome (auteur d'origine)
- [Kendzione](https://github.com/Kendzione/esphome-victron_ble) — fork du composant BLE compatible ESPHome ≥ 2026.11
- [KinDR007](https://github.com/KinDR007/VictronMPPT-ESPHOME) — composant VE.Direct pour ESPHome
- [mr-manuel](https://github.com/mr-manuel/venus-os_dbus-mqtt-solar-charger) — bridge MQTT → dbus Venus OS (solar charger)
- [freakent](https://github.com/freakent/dbus-mqtt-devices) — bridge MQTT → dbus Venus OS (devices, v6.5)
