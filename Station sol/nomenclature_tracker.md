# Station sol – Tracker d'antenne : architecture et nomenclature

Version 2, à valider. Prévue pour **3 trackers**. Les prix sont des ordres de grandeur et les références citées de mémoire doivent être vérifiées chez le distributeur (Mouser, DigiKey, LCSC) avant de commander.

## Choix retenus

| Sujet | Choix |
|---|---|
| Vidéo de la fusée | Émetteur **TX5813** (5,8 GHz, 20 mW) + caméra FPV analogique |
| Réception vidéo | **Boîtier RC832 / RC832H** sur la tête, derrière l'antenne 16 dBi. **Aucune électronique vidéo sur le PCB** |
| Vidéo vers le PC | Câble AV (jaune) → **boîtier d'acquisition USB** (convertisseur « VHS vers numérique », sinon August VGB100) → OBS ou VLC |
| Télémétrie | LoRa **868 MHz**, **RFM95W** comme sur Sweeton |
| Suivi | La fusée envoie sa **position GNSS** ; la station calcule l'azimut et l'élévation |
| Diversité vidéo | **Non** (coût). Une seule antenne vidéo par tracker |
| Alimentation | **Batterie 12 V LiFePO4** (voir plus bas) |
| PC | 2 câbles USB : télémétrie (port série virtuel du STM32) et vidéo (boîtier d'acquisition) |

## Architecture

```
                 TÊTE ORIENTABLE (tourne avec les antennes)
   ┌──────────────────────────────────────────────────────────────┐
   │ Antenne 16 dBi ─(SMA court)─► RC832 ─► câble AV jaune        │
   │ Antenne 868 MHz ─(coax court)─► vers carte (RFM95W)          │
   │ Encodeur d'élévation AS5048A                                 │
   └──────────────┬───────────────────────────────────────────────┘
                  │ 12 V du RC832, vidéo AV, SPI encodeur, coax 868 MHz
                  │ (azimut limité à ±270° avec une boucle de câble)
   ┌──────────────┴───────────────────────────────────────────────┐
   │ BASE : carte du tracker                                      │
   │ STM32H523 ── USB-C ──► PC (télémétrie en direct)             │
   │   ├─ RFM95W (télémétrie de la fusée)                         │
   │   ├─ GNSS MAX-M10S (position de la station)                  │
   │   ├─ Magnétomètre (nord au démarrage) + IMU (niveau)         │
   │   ├─ Encodeur d'azimut AS5048A                               │
   │   ├─ 2× TMC2209 ─► 2× NEMA 17 + réduction                    │
   │   └─ Écran OLED, boutons, buzzer, LED                        │
   │ Batterie 12 V ─► protections ─► 12 V moteurs + sortie RC832  │
   │                              └► buck 5 V ─► LDO 3,3 V        │
   └──────────────────────────────────────────────────────────────┘
   Câble AV jaune du RC832 ─► boîtier d'acquisition USB ─► PC
```

**Principe du suivi :**
1. Au démarrage : position GNSS de la station, nord (magnétomètre ou visée manuelle), mise à zéro des axes avec les encodeurs.
2. À chaque trame de télémétrie : calcul de l'azimut et de l'élévation vers la fusée, commande des moteurs.
3. Entre deux trames : extrapolation de la trajectoire avec la vitesse de la fusée.

## Nomenclature de la carte du tracker (par tracker)

### 1. Processeur et liaison PC

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Microcontrôleur | **STM32H523RET6** (LQFP64, 250 MHz, USB) | 1 | ~5 € | Même famille que Sweeton : drivers RFM95W et GNSS réutilisés |
| Quartz HSE | 8 ou 25 MHz, boîtier 3225, + 2 condensateurs | 1 | ~0,30 € | Horloge précise pour l'USB |
| Connecteur USB | USB-C 16 broches (ex. GCT USB4105-GF-A) | 1 | ~0,80 € | Télémétrie vers le PC |
| Protection ESD USB | **USBLC6-2SC6** (ST) | 1 | ~0,40 € | Protège D+ / D− |
| Résistances CC | 5,1 kΩ | 2 | — | Obligatoires en USB-C côté périphérique |
| Debug | Connecteur SWD 2×5 au pas de 1,27 mm | 1 | ~1 € | Programmation |
| Boutons reset / BOOT0 | **C&K PTS810 SJM 250 SMTR LFS** | 2 | ~0,30 € | — |

### 2. Radio de télémétrie

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Module LoRa 868 MHz | **RFM95W-868S2** (HopeRF) | 1 | ~6–8 € | Identique à la fusée |
| Connecteurs antenne | **Molex 73251-1150** (SMA, bord de carte) | 2 | ~3–5 € | Radio et GNSS |
| Adaptation | Réseau en π (3 pastilles 0402) | 1 | — | Réglage sans refaire la carte |
| Antenne 868 MHz | Yagi ou patch directive 868 MHz, SMA | 1 | ~20–40 € | Sur la tête orientable |

### 3. Position et orientation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| GNSS | **u-blox MAX-M10S** | 1 | ~10–15 € | Station fixe : quelques mètres suffisent |
| Antenne GNSS | Patch actif 1575 MHz, SMA | 1 | ~5–10 € | — |
| Encodeurs d'axes | **AS5048A** (14 bits, SPI) + aimant diamétral 6 mm | 2 | ~5–8 € | Angle absolu de chaque axe. Celui d'élévation est sur une petite carte fille sur la tête |
| Magnétomètre | **MMC5983MA** ou **LIS3MDL** | 1 | ~2–4 € | Nord au démarrage. À 15–20 cm des moteurs |
| IMU | **LSM6DSO** (ST) | 1 | ~3 € | Mise à niveau de la base |

### 4. Motorisation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Moteurs | **NEMA 17**, ~40 N·cm, 1,5–1,7 A (ex. 17HS4401) | 2 | ~10–15 € | — |
| Réduction | Vis sans fin ou courroie GT2 (1:5 à 1:10) | 2 | ~10–20 € | La vis sans fin bloque la tête sans courant |
| Drivers | **TMC2209** (ADI Trinamic) | 2 | ~3–5 € | Silencieux, courant réglable par UART |
| Condensateurs moteurs | 100 µF / 35 V électrolytique + 100 nF | 2 | ~0,50 € | Un par driver |
| Connecteurs moteurs | JST-XH 4 broches | 2 | ~0,30 € | — |

### 5. Alimentation

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Connecteur batterie | **XT60** (mâle sur la carte) | 1 | ~1 € | — |
| Fusible | Fusible lame 5 A ou réarmable | 1 | ~0,50 € | — |
| Protection d'inversion | MOSFET P **AO4407A** | 1 | ~0,50 € | Batterie branchée à l'envers sans dégât |
| Protection surtension | TVS **SMBJ24A** | 1 | ~0,30 € | Pics dus aux moteurs |
| Interrupteur marche/arrêt | **JS202011SCQN** (C&K) + MOSFET P de puissance | 1 | ~1 € | La glissière commande le MOSFET, pas le courant directement |
| Sortie 12 V du RC832 | Connecteur JST-XH 2 broches + fusible réarmable 500 mA + ferrite + 100 µF | 1 | ~1 € | Filtre les parasites des moteurs, sinon l'image est brouillée |
| 12 V → 5 V | Buck **TPS62933** (TI, 3 A) | 1 | ~1,50 € | Électronique, écran |
| 5 V → 3,3 V | **TLV1117LV33** (SOT-223) | 1 | ~0,50 € | MCU, radio, GNSS |
| 3,3 V capteurs (option) | **TPS7A2033** (faible bruit) | 1 | ~0,80 € | Encodeurs, magnétomètre, IMU |
| Mesure de la batterie | Pont diviseur 100 kΩ / 10 kΩ + 100 nF vers l'ADC | 1 | ~0,10 € | Alerte batterie faible. INA228 si tu veux aussi le courant |

### 6. Interface

| Fonction | Composant | Qté | Prix env. | Remarque |
|---|---|---|---|---|
| Écran | OLED 1,3" I²C (SH1106) | 1 | ~5 € | État GNSS, télémétrie, batterie, angles |
| Boutons | **C&K PTS810 SJM 250 SMTR LFS** | 3 | ~0,50 € | Menu, mise à zéro |
| Buzzer | **CMT-8530S-SMT-TR** + MOSFET 2N7002 + diode BAT54 | 1 | ~1,50 € | Perte de télémétrie, batterie faible |
| LED | Alimentation, état, télémétrie | 3 | ~0,30 € | — |

## Matériel hors carte (par tracker)

| Élément | Référence | Prix env. |
|---|---|---|
| Antenne vidéo 16 dBi | Déjà en ta possession (vérifier bande, connecteur SMA/RP-SMA et polarisation) | — |
| Récepteur vidéo | **RC832 / RC832H** 48 canaux | ~20–40 € |
| Boîtier d'acquisition | Convertisseur USB « VHS vers numérique » (puce MS2106/MS2107 ou UTV007) ou **August VGB100** | ~10–35 € |
| Batterie | **LiFePO4 12,8 V, 6 à 10 Ah** avec BMS intégré | ~35–60 € |
| Chargeur | Chargeur LiFePO4 14,6 V | ~15–25 € |
| Câbles | SMA court pour la 16 dBi, câble AV 2–3 m, USB | ~10 € |

## Alimentation 12 V : d'où la sortir

**Recommandé : une batterie LiFePO4 12,8 V (4S) de 6 à 10 Ah par tracker.**
- Tension stable (12,8 V nominal, 14,6 V pleine, ~11 V vide), compatible avec les TMC2209 (4,75–29 V) et le RC832.
- Beaucoup plus sûre qu'une LiPo sur le terrain : pas d'emballement thermique, BMS intégré.
- **Consommation estimée** : ~20–25 W en pointe (moteurs en mouvement), ~8–10 W en moyenne. Une 10 Ah (~128 Wh) tient environ **10 heures**, une 6 Ah environ 6 heures.

| Autre source | Avis |
|---|---|
| Batterie plomb 12 V 7 Ah | Fonctionne, pas chère, mais lourde (~2 kg) |
| LiPo 3S (11,1 V) | Fonctionne, mais fragile et dangereuse si elle est abîmée. Éviter les 4S (16,8 V) sans vérifier la tension max du RC832 |
| Bloc secteur 12 V / 5 A (60 W) | Parfait pour les essais à l'atelier, rarement possible sur un terrain de lancement |
| Batterie de voiture ou prise allume-cigare | Possible avec un câble adapté, mais la tension varie (jusqu'à 14,5 V moteur tournant) : la protection TVS est indispensable |

## Points de conception importants

1. **Câbles de la tête.** Seuls le 12 V du RC832, la vidéo AV, le SPI de l'encodeur d'élévation et le coax 868 MHz montent vers la tête. Limite l'azimut à environ ±270° avec une boucle de câble : pas besoin de collecteur tournant.
2. **Magnétomètre et moteurs.** Les aimants des moteurs faussent le compas. Ne l'utilise que pour trouver le nord au démarrage ; pendant le suivi, ce sont les encodeurs qui donnent l'angle.
3. **Réglementation.** 5,8 GHz analogique limité à 25 mW PIRE en France (le TX5813 fait 20 mW). LoRa 868 MHz : limites de puissance et de cycle d'émission.
4. **Débit de télémétrie.** Trames courtes (position, vitesse, état) ; la station extrapole entre deux trames.
5. **Précision de pointage.** L'antenne 16 dBi a un faisceau d'environ 30° : une précision de ±5° suffit.
6. **Récepteur au plus près de l'antenne.** À 5,8 GHz, un câble coaxial fin perd 1,5–2 dB par mètre : le RC832 est fixé juste derrière l'antenne.

## Portée vidéo estimée

Émetteur 20 mW, antenne omni ~2 dBi sur la fusée, 16 dBi au sol, sans marge :

| Distance | Puissance reçue |
|---|---|
| 2 km | ~−83 dBm |
| 5 km | ~−91 dBm |
| 10 km | ~−97 dBm |

Avec une sensibilité d'environ −90 dBm, compte sur **2 à 4 km d'image propre**. Les creux de l'antenne de la fusée retirent facilement 10 dB.

## Évolution possible : diversité vidéo

Non retenue pour la première version (coût). À terme : un second récepteur sur une antenne omni pour les premières secondes après le décollage, quand la fusée passe trop vite pour le tracker.

## Questions ouvertes

- Antenne 16 dBi : bande exacte, connecteur (SMA ou RP-SMA), polarisation (RHCP ou LHCP) ?
- Portée et altitude maximales visées ?
- Système d'exploitation du PC de télémétrie (Windows ou Linux) ?
