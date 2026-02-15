# Victron Unified ESP32 → Venus OS (MQTT)
## README_Venus.md

## 🎯 Objectif
Ce projet fournit une implémentation GX-like complète basée sur ESPHome + ESP32, permettant de connecter des équipements Victron sans Cerbo GX, tout en restant pleinement compatible Venus OS via MQTT.

Il agrège BLE + VE.Direct de façon redondante pour :
- 1 × SmartShunt 500A
- 2 × MPPT Victron (100/20 + 100/30)

et expose ces données à :
- Home Assistant
- Venus OS (ESS / DVCC / DBus MQTT)

## 🧱 Architecture générale

### Équipements supportés
| Équipement | BLE | VE.Direct | Rôle |
|----------|-----|-----------|------|
| SmartShunt 500A | ✅ | ✅ | Batterie principale |
| MPPT 100/20 | ✅ | ✅ | Chargeur solaire |
| MPPT 100/30 | ✅ | ✅ | Chargeur solaire |

### Philosophie
- BLE prioritaire (temps réel)
- VE.Direct en fallback (robustesse)
- Publication Venus-compatible (DBus MQTT)

## 🔋 Intégration Batterie (SmartShunt → Venus)

Topic :
device/victron_unified/Proxy

Données publiées :
- Dc/0/Voltage
- Dc/0/Current
- Dc/0/Power
- Soc
- ConsumedAh
- InstalledCapacity
- TimeToGo (min)

Historique journalier :
- MinVoltage
- MaxVoltage

## ☀️ Intégration MPPT → Venus OS

Topics :
- venus/solarcharger/mppt_100_20
- venus/solarcharger/mppt_100_30

Fonctions reconstruites :
- Yield Today / Yesterday / Lifetime
- Max Power Today
- Max PV Voltage Today
- Time Bulk / Absorption / Float

## 🚨 Gestion des erreurs MPPT
Les ErrorCode sont remontés dynamiquement depuis VE.Direct (plus de forçage à 0).

## ⚙️ ESS / DVCC
Contrôle :
- ESS Mode
- Minimum SOC
- Battery Power Setpoint
- Charge Current Limit
- MPPT Enable/Disable
- DVCC Allow Charge

## 🟢 Keepalive GX
- Status publié régulièrement
- Keepalive envoyé toutes les 40s

## ⚠️ Limitations
- ChargeCurrentLimit affiché statiquement côté GX
- PortalId fixe
- Historique limité à J / J-1

## 🏁 Conclusion
Solution GX-like avancée, stable et redondante, permettant une intégration Venus OS complète sans matériel GX officiel.

Projet non officiel Victron Energy.
