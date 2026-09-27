# Station sol – Tracker d'antenne : architecture et nomenclature

Première version, à valider. Les prix sont des ordres de grandeur et les références citées de mémoire doivent être vérifiées chez le distributeur (Mouser, DigiKey, LCSC) avant de commander.

## Hypothèses de départ

| Sujet | Hypothèse retenue | À confirmer |
|---|---|---|
| Vidéo de la fusée | Analogique FPV en **5,8 GHz** | Bande exacte et type d'antenne que tu as déjà |
| Télémétrie | LoRa **868 MHz** avec un RFM95W, comme sur Sweeton | Modulation et paramètres LoRa côté fusée |
| Suivi | La fusée envoie sa **position GNSS** par télémétrie ; la station connaît la sienne et calcule l'azimut et l'élévation | — |
| Alimentation | Batterie terrain **12 V** (LiFePO4 4S ou plomb) ou bloc secteur 12 V / 5 A | — |
| PC | Liaison **USB** : télémétrie en port série virtuel (CDC), vidéo par ta carte de capture HDMI → USB existante | — |

## Architecture

```
                    TÊTE ORIENTABLE (tourne avec les antennes)
   ┌───────────────────────────────────────────────────────────────┐
   │ Antenne vidéo 5,8 GHz (directive) ─► RX5808 ─► vidéo CVBS      │
   │ Antenne 868 MHz (directive)       ─► RFM95W (SPI)              │
   │ IMU (inclinaison) + encodeur magnétique d'élévation            │
   └──────────────┬────────────────────────────────────────────────┘
                  │ câbles : alimentation, SPI/UART, vidéo CVBS
                  │ (enroulement limité à ±270° ou collecteur tournant)
   ┌──────────────┴────────────────────────────────────────────────┐
   │ BASE                                                          │
   │ STM32H523 ── USB-C ──► PC (télémétrie en direct)              │
   │   ├─ GNSS (position de la station)                            │
   │   ├─ Magnétomètre (nord, calibré au démarrage)                │
   │   ├─ Encodeur magnétique d'azimut                             │
   │   ├─ 2× drivers TMC2209 ─► 2× moteurs NEMA 17 + réduction     │
   │   └─ Écran, boutons, buzzer                                   │
   │ Vidéo CVBS ─► convertisseur AV → HDMI ─► ta carte de capture  │
   │ Alimentation 12 V ─► protection ─► buck 5 V ─► LDO 3,3 V      │
   └───────────────────────────────────────────────────────────────┘
```

**Principe du suivi :**
1. Au démarrage, la station lit sa position GNSS, trouve le nord (magnétomètre et/ou visée manuelle) et fait la mise à zéro des axes (encodeurs ou butées).
2. À chaque trame de télémétrie, elle calcule l'azimut et l'élévation de la fusée à partir des deux positions GNSS, puis commande les moteurs.
3. Entre deux trames, elle extrapole la trajectoire avec la vitesse de la fusée. Le **RSSI** de la vidéo et de la radio sert à affiner le pointage ou à chercher la fusée si la télémétrie est perdue.

## Nomenclature

### 1. Processeur et liaison PC

| Fonction | Composant | Qté | Prix env. | Pourquoi |
|---|---|---|---|---|
| Microcontrôleur | **STM32H523RET6** (LQFP64, 250 MHz, USB) | 1 | ~5 € | Même famille que Sweeton : drivers RFM95W, GNSS et outils réutilisés. 49 E/S suffisent ici |
| Quartz HSE | 8 ou 25 MHz, 3225 | 1 | ~0,30 € | L'USB demande une horloge précise |
| Connecteur USB | USB-C 16 broches (ex. GCT USB4105-GF-A) | 1 | ~0,80 € | Liaison PC, alimentation de debug |
| Protection ESD USB | **USBLC6-2SC6** (ST) | 1 | ~0,40 € | Protège D+ / D− |
| Résistances CC | 2× 5,1 kΩ | 2 | — | Obligatoires en USB-C côté périphérique |
| Debug | Connecteur SWD (Tag-Connect ou 2×5 au pas de 1,27 mm) | 1 | — | Programmation |

### 2. Réception vidéo

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Récepteur 5,8 GHz | **Module RX5808** (puce RTC6715), version avec SPI activé | 1 ou 2 | ~8–12 € | Sort la vidéo composite (CVBS) et une tension **RSSI** analogique. Le canal se choisit par SPI |
| Commutation de diversité (option) | **TS5V330** (TI) | 1 | ~1 € | Si 2 récepteurs (antenne directive + omni) : choisit la meilleure image selon le RSSI |
| Buffer vidéo 75 Ω | **THS7314** (TI) ou équivalent | 1 | ~1,50 € | Attaque le câble vers le convertisseur |
| Connecteur vidéo | RCA femelle pour CI ou SMA | 1 | ~0,50 € | Sortie CVBS |
| Convertisseur vers le PC | **Convertisseur AV (CVBS) → HDMI** (« AV2HDMI ») | 1 | ~10–15 € | Se branche sur ta carte de capture HDMI → USB existante |

Alternative plus simple pour la vidéo : une **carte d'acquisition USB composite (UVC)**, reconnue comme une webcam, qui évite le passage par HDMI.

### 3. Radio de télémétrie

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Module LoRa 868 MHz | **RFM95W-868S2** (HopeRF) | 1 | ~6–8 € | Identique à la fusée, donc mêmes réglages et même driver |
| Connecteur antenne | **Molex 73251-1150** (SMA, bord de carte) | 2 | ~3–5 € | Un pour la radio, un pour le GNSS |
| Antenne 868 MHz | Yagi ou patch directive 868 MHz, SMA | 1 | ~20–40 € | Sur la tête orientable. Gros gain de portée par rapport à une antenne omni |
| Adaptation | Réseau en π (3 pastilles 0402) | 1 | — | Réglage de l'antenne sans refaire la carte |

### 4. Position et orientation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| GNSS | **u-blox MAX-M10S** | 1 | ~10–15 € | La station est fixe : quelques mètres de précision suffisent face à une fusée à plusieurs km |
| Antenne GNSS | Patch actif 1575 MHz, SMA | 1 | ~5–10 € | — |
| Encodeurs d'axes | **AS5048A** (ams OSRAM, 14 bits, SPI) + aimant diamétral | 2 | ~5–8 € | Mesure **absolue** de l'angle de chaque axe : pas de dérive si un moteur saute des pas |
| Magnétomètre (compas) | **MMC5983MA** (Memsic) ou **LIS3MDL** (ST) | 1 | ~2–4 € | Trouve le nord au démarrage. **Éloigne-le des moteurs** et calibre-le sur place |
| IMU / inclinomètre | **LSM6DSO** ou ISM330DHCX (ST) | 1 | ~3–8 € | Mesure l'inclinaison de la tête et vérifie que la base est de niveau |
| Fins de course (option) | Capteur à effet Hall ou micro-rupteur | 2 | ~1 € | Mise à zéro de secours |

### 5. Motorisation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Moteurs | **NEMA 17**, ~40 N·cm, 1,5–1,7 A (ex. 17HS4401) | 2 | ~10–15 € | Standard, facile à trouver |
| Réduction | Vis sans fin ou courroie GT2 (rapport 1:5 à 1:10) | 2 | ~10–20 € | Plus de couple et de précision. Une vis sans fin **bloque** la tête sans courant (bien contre le vent) |
| Drivers | **TMC2209** (ADI Trinamic) | 2 | ~3–5 € la puce | Silencieux, réglage du courant par UART, détection de blocage (StallGuard) pour une mise à zéro sans capteur |
| Découplage moteurs | 100 µF / 35 V électrolytique + 100 nF par driver | 2 | — | Absorbe les pics de courant |
| Connecteurs moteurs | JST-XH 4 broches ou borniers | 2 | — | — |

### 6. Alimentation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Connecteur d'entrée | **XT60** ou jack DC 5,5 mm | 1 | ~1 € | 12 V |
| Fusible | Fusible réarmable ou lame 5 A | 1 | ~0,50 € | — |
| Protection d'inversion | MOSFET P (ex. AO4407A) ou contrôleur de diode idéale | 1 | ~0,50 € | Peu de pertes, contrairement à une diode série |
| Protection surtension | TVS **SMBJ24A** | 1 | ~0,30 € | Pics dus aux moteurs |
| 12 V → 5 V | Buck **TPS62933** (TI, 3 A) | 1 | ~1,50 € | Électronique, récepteur vidéo et écran |
| 5 V → 3,3 V numérique | **TLV1117LV33** (SOT-223) | 1 | ~0,50 € | MCU, radio, GNSS |
| 3,3 V capteurs (option) | **TPS7A2033** (faible bruit) | 1 | ~0,80 € | Encodeurs, magnétomètre, IMU |
| Mesure de la batterie | **INA228** + shunt 2 mΩ, ou pont diviseur | 1 | ~3 € | Alerte batterie faible sur le terrain |

Les moteurs sont alimentés **directement en 12 V** par les drivers TMC2209 (plage de 4,75 à 29 V).

### 7. Interface

| Fonction | Composant | Qté | Remarque |
|---|---|---|---|
| Écran | OLED 0,96" ou 1,3" en I²C (SSD1306 / SH1106) | 1 | Canal vidéo, RSSI, état GNSS et télémétrie, tension batterie |
| Boutons | **C&K PTS810 SJM 250 SMTR LFS** | 3–4 | Navigation, choix du canal, mise à zéro |
| Buzzer | **CMT-8530S-SMT-TR** (+ MOSFET + diode) | 1 | Alerte perte de signal ou batterie faible |
| LED | État, alimentation | 2–3 | — |

## Points de conception importants

1. **Câbles de la tête orientable.** Un collecteur tournant ne transmet pas proprement du 5,8 GHz. Place donc **les récepteurs sur la tête**, à côté des antennes, et ne fais passer que l'alimentation, les données (SPI/UART) et la vidéo composite. Soit tu limites l'azimut à environ ±270° avec une boucle de câble, soit tu utilises un collecteur tournant à 6–12 voies.
2. **Magnétomètre et moteurs.** Les aimants des moteurs pas à pas faussent complètement le compas. Mets-le à au moins 15–20 cm des moteurs, et ne l'utilise que **pour trouver le nord au démarrage**. Pendant le suivi, ce sont les encodeurs qui donnent l'angle.
3. **Réglementation.** En France, la vidéo analogique en 5,8 GHz est limitée à **25 mW PIRE** côté fusée, et la LoRa 868 MHz est soumise aux limites de puissance et de cycle d'émission. Les bandes 1,2 / 1,3 GHz demandent une licence radioamateur.
4. **Débit de télémétrie.** Avec le cycle d'émission de 1 % en 868 MHz, envoie des trames courtes (position, vitesse, état). La station extrapole la trajectoire entre deux trames.
5. **Précision de pointage.** Une antenne 5,8 GHz directive a un faisceau d'environ 20–30°. Une précision de ±2–3° suffit, donc la réduction et les encodeurs sont largement assez précis.

## Questions ouvertes

- Quelle est l'antenne que tu as déjà : bande, type (patch, hélice, Yagi), connecteur ?
- Quelle bande et quel émetteur vidéo sont dans la fusée ?
- Portée et altitude maximales visées ?
- Alimentation : batterie terrain ou secteur ?
- Budget global de la station ?
