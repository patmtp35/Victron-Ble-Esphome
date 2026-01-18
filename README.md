Victron Unified ESP – BLE + VE.Direct redondant

2× SmartSolar MPPT (100/20 + 100/30) + SmartShunt 500A

⚠️ VERSION PERSONNELLE – adaptée à mes besoins
Ce projet n’est pas un template générique, mais une configuration ESPHome avancée, pensée pour la résilience, la continuité de service et la compatibilité Home Assistant existante.

📌 Description

Projet ESPHome permettant de lire simultanément et en parallèle les données Victron via :

Bluetooth Low Energy (BLE)

VE.Direct (UART)

Redondance automatique BLE ↔ VE.Direct selon la disponibilité

Normalisation et compatibilité rétro Home Assistant

Templates unifiés (“BEST source”) pour exposer la meilleure donnée

Support multi-MPPT + SmartShunt sur un seul ESP32

⚠️ En BLE, Victron ne fournit pas toutes les métriques disponibles en VE.Direct.
Le BLE est utilisé comme source rapide et redondante, le VE.Direct comme référence complète et stable.

🔧 Matériel compatible

Victron SmartSolar MPPT 100/20

Victron SmartSolar MPPT 100/30

Victron SmartShunt 500A

ESP32 (BLE + 3 UART)

Home Assistant + ESPHome

✨ Fonctionnalités principales
✔ Télémetrie BLE (temps réel)

Lecture directe via BLE :

Tension batterie

Courant batterie

Puissance batterie / PV

Tension panneau

Courant Load

Température MPPT

Production du jour (Yield today)

SOC (SmartShunt)

👉 BLE = rapide, sans fil, mais jeu de données partiel

✔ Télémetrie VE.Direct (complète & fiable)

Lecture UART VE.Direct :

Panel voltage / power

Battery voltage / current

Load current

Yield today / yesterday / total

Max power today / yesterday

Charging mode / tracking mode

Error codes

Firmware, type d’appareil, numéro de série

Consumed Ah (SmartShunt)

👉 VE.Direct = référence complète, historique fiable

✔ Redondance automatique (BEST source)

Les capteurs critiques utilisent des templates intelligents :

Priorité BLE si disponible

Fallback VE.Direct automatique si BLE absent ou invalide

Calculs de secours (ex: puissance = V × I si nécessaire)

➡️ Aucune interruption côté Home Assistant
➡️ Les dashboards et automatisations restent stables

🧠 Architecture générale

BLE actif en permanence (MPPT + SmartShunt)

3 UART dédiés, un par appareil VE.Direct

Aucun partage d’UART (protocole VE.Direct half-duplex)

Les capteurs VE.Direct sont majoritairement internal

Les entités exposées HA sont unifiées et stables

🔌 Câblage VE.Direct (double MPPT + SmartShunt)

🎯 Objectif :
Connecter 2 MPPT + 1 SmartShunt sur un seul ESP32, proprement et sans conflit.

🔥 Règle essentielle

❌ Impossible de partager un UART entre deux VE.Direct
✔ Chaque appareil Victron = son propre UART

📍 Mapping UART réel (aligné avec le code)
Appareil	RX ESP32	TX ESP32	UART
MPPT 100/20	GPIO19	GPIO18 (NC)	UART
MPPT 100/30	GPIO16	GPIO17 (NC)	UART
SmartShunt 500A	GPIO22	GPIO21 (NC)	UART

✔ TX Victron → RX ESP32
✔ TX ESP32 non utilisé
✔ Masse commune (GND) obligatoire
✔ Câbles courts, torsadés recommandés

🧩 Schéma logique simplifié
+-----------------------+
|        ESP32          |
|                       |
| UART RX19  ← MPPT100/20
| UART RX16  ← MPPT100/30
| UART RX22  ← SmartShunt
|                       |
+-----------------------+
        |      |      |
       GND    GND    GND

🧩 Home Assistant & compatibilité

Les anciens capteurs smartsolar1_esp_* sont recréés via templates

Aucun dashboard ou automation existant n’est cassé

Les nouvelles entités BLE / VE.Direct peuvent coexister

Bouton restart conservé pour compatibilité

🧪 Philosophie du projet

Résilience avant tout

Pas de dépendance à une seule techno

BLE = continuité

VE.Direct = précision

ESPHome lisible, maintenable, évolutif

🤝 Ressources & crédits

ESPHome Victron BLE – Fabian Schmidt
https://github.com/Fabian-Schmidt/esphome-victron_ble

ESPHome Victron VE.Direct – KinDR007
https://github.com/KinDR007/VictronMPPT-ESPHOME

🙏 Merci à eux pour le travail fondamental sur lequel repose ce projet.
