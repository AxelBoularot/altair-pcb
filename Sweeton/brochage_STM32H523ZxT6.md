# Sweeton – Brochage du STM32H523ZxT6 (LQFP144)

Affectation des 112 E/S du STM32H523ZET6 / ZCT6 : 2 broches de debug SWD et 110 GPIO pour les fonctions de l'ordinateur de vol.

Vérifications faites automatiquement à partir des données officielles ST ([STM32_open_pin_data](https://github.com/STMicroelectronics/STM32_open_pin_data), fichier `STM32H523ZETx.xml`) :

- chaque fonction alternative (SPI, UART, TIM…) existe bien sur la broche choisie ;
- aucune broche n'est utilisée deux fois ;
- chaque entrée d'interruption a sa propre ligne EXTI (0 à 15), sans conflit.

Le ZET6 (512 Ko de flash) et le ZCT6 (256 Ko) ont le même brochage : ce fichier vaut pour les deux.

À valider dans STM32CubeMX avant le routage, notamment les numéros d'AF et les contraintes de vitesse des broches.

## Résumé

| Groupe | Broches |
|---|---|
| Système | 9 |
| IMU principale (SPI1) | 5 |
| IMU secondaire (SPI2) | 7 |
| Baro + magnéto (SPI3) | 7 |
| Flash de log (OCTOSPI1) | 6 |
| Carte microSD (SDMMC1) | 7 |
| CAN-FD | 8 |
| GNSS (USART3) | 4 |
| Radio télémétrie (USART2) | 5 |
| Superviseur sécurité (USART6) | 7 |
| Actionneurs (PWM) | 9 |
| Sorties de puissance | 6 |
| Bus d'extension | 13 |
| I2C interne | 2 |
| Mesures analogiques (ADC) | 12 |
| Interface | 5 |
| **Total** | **112** |

Broches dédiées hors GPIO : `BOOT0` (138), `NRST` (25), `VBAT`, `VCAP`, `VDD`/`VSS`, `VDDA`/`VSSA`/`VREF+`, `VDDUSB`, `VDDIO2`.

## Système

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PH0 | 23 | HSE_IN | RCC_OSC_IN | in |  |
| PH1 | 24 | HSE_OUT | RCC_OSC_OUT | out |  |
| PA13 | 105 | SWDIO | GPIO | io |  |
| PA14 | 109 | SWCLK | GPIO | in |  |
| PA11 | 103 | USB_DM | USB_DM | io |  |
| PA12 | 104 | USB_DP | USB_DP | io |  |
| PC13 | 7 | USB_VBUS_DETECT | GPIO | in | EXTI13 |
| PA9 | 101 | DEBUG_TX (console) | USART1_TX | out |  |
| PA10 | 102 | DEBUG_RX (console) | USART1_RX | in |  |

## IMU principale (SPI1)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PA5 | 41 | IMU1_SCK | SPI1_SCK | out |  |
| PA6 | 42 | IMU1_MISO | SPI1_MISO | in |  |
| PA7 | 43 | IMU1_MOSI | SPI1_MOSI | out |  |
| PA4 | 40 | IMU1_CS | GPIO | out |  |
| PG0 | 56 | IMU1_INT1 | GPIO | in | EXTI0 |

## IMU secondaire (SPI2)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PB13 | 74 | IMU2_SCK | SPI2_SCK | out |  |
| PB14 | 75 | IMU2_MISO | SPI2_MISO | in |  |
| PB15 | 76 | IMU2_MOSI | SPI2_MOSI | out |  |
| PB12 | 73 | IMU2_CS_ACC | GPIO | out |  |
| PB10 | 69 | IMU2_CS_GYR | GPIO | out |  |
| PG2 | 87 | IMU2_INT_ACC | GPIO | in | EXTI2 |
| PG3 | 88 | IMU2_INT_GYR | GPIO | in | EXTI3 |

## Baro + magnéto (SPI3)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PB3 | 133 | SENS_SCK | SPI3_SCK | out |  |
| PB4 | 134 | SENS_MISO | SPI3_MISO | in |  |
| PG8 | 93 | SENS_MOSI | SPI3_MOSI | out |  |
| PD10 | 79 | BARO_CS | GPIO | out |  |
| PD11 | 80 | MAG_CS | GPIO | out |  |
| PG4 | 89 | BARO_INT | GPIO | in | EXTI4 |
| PG5 | 90 | MAG_DRDY | GPIO | in | EXTI5 |

## Flash de log (OCTOSPI1)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PF10 | 22 | FLASH_CLK | OCTOSPI1_CLK | out |  |
| PG6 | 91 | FLASH_NCS | OCTOSPI1_NCS | out |  |
| PF8 | 20 | FLASH_IO0 | OCTOSPI1_IO0 | io |  |
| PF9 | 21 | FLASH_IO1 | OCTOSPI1_IO1 | io |  |
| PF7 | 19 | FLASH_IO2 | OCTOSPI1_IO2 | io |  |
| PF6 | 18 | FLASH_IO3 | OCTOSPI1_IO3 | io |  |

## Carte microSD (SDMMC1)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PC12 | 113 | SD_CK | SDMMC1_CK | out |  |
| PD2 | 116 | SD_CMD | SDMMC1_CMD | io |  |
| PC8 | 98 | SD_D0 | SDMMC1_D0 | io |  |
| PC9 | 99 | SD_D1 | SDMMC1_D1 | io |  |
| PC10 | 111 | SD_D2 | SDMMC1_D2 | io |  |
| PC11 | 112 | SD_D3 | SDMMC1_D3 | io |  |
| PG15 | 132 | SD_DETECT | GPIO | in |  |

## CAN-FD

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PD0 | 114 | CAN1_RX | FDCAN1_RX | in |  |
| PD1 | 115 | CAN1_TX | FDCAN1_TX | out |  |
| PE0 | 141 | CAN1_STBY | GPIO | out |  |
| PF2 | 12 | CAN1_TERM_EN (120 Ω) | GPIO | out |  |
| PB5 | 135 | CAN2_RX | FDCAN2_RX | in |  |
| PB6 | 136 | CAN2_TX | FDCAN2_TX | out |  |
| PB7 | 137 | CAN2_STBY | GPIO | out |  |
| PF3 | 13 | CAN2_TERM_EN (120 Ω) | GPIO | out |  |

## GNSS (USART3)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PD8 | 77 | GNSS_TX | USART3_TX | out |  |
| PD9 | 78 | GNSS_RX | USART3_RX | in |  |
| PA15 | 110 | GNSS_PPS (capture TIM2 32 bits) | TIM2_CH1 | in |  |
| PG12 | 127 | GNSS_RESET_N | GPIO | out |  |

## Radio télémétrie (USART2)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PA2 | 36 | RADIO_TX | USART2_TX | out |  |
| PA3 | 37 | RADIO_RX | USART2_RX | in |  |
| PD3 | 117 | RADIO_CTS | USART2_CTS | in |  |
| PD4 | 118 | RADIO_RTS | USART2_RTS | out |  |
| PD5 | 119 | RADIO_EN | GPIO | out |  |

## Superviseur sécurité (USART6)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PG14 | 129 | SUP_TX | USART6_TX | out |  |
| PG9 | 124 | SUP_RX | USART6_RX | in |  |
| PG13 | 128 | SUP_HEARTBEAT | GPIO | out |  |
| PE12 | 65 | SUP_KILL_N | GPIO | in | EXTI12 |
| PG10 | 125 | ARMED_STATUS | GPIO | in |  |
| PE15 | 68 | ARM_SWITCH | GPIO | in | EXTI15 |
| PF4 | 14 | ACT_PWR_EN (rail servos) | GPIO | out |  |

## Actionneurs (PWM)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PE9 | 60 | PWM1 – TVC tangage | TIM1_CH1 | out |  |
| PE11 | 64 | PWM2 – TVC lacet | TIM1_CH2 | out |  |
| PE13 | 66 | PWM3 – vanne de throttle | TIM1_CH3 | out |  |
| PE14 | 67 | PWM4 – réserve | TIM1_CH4 | out |  |
| PD12 | 81 | PWM5 – roulis / RCS | TIM4_CH1 | out |  |
| PD13 | 82 | PWM6 – roulis / RCS | TIM4_CH2 | out |  |
| PD14 | 85 | PWM7 – aux | TIM4_CH3 | out |  |
| PD15 | 86 | PWM8 – aux | TIM4_CH4 | out |  |
| PC7 | 97 | IMU_HEATER (chauffe IMU) | TIM3_CH2 | out |  |

## Sorties de puissance

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PE3 | 2 | OUT1_EN (vanne solénoïde) | GPIO | out |  |
| PE7 | 58 | OUT2_EN | GPIO | out |  |
| PF5 | 15 | OUT3_EN | GPIO | out |  |
| PF15 | 55 | OUT4_EN | GPIO | out |  |
| PB2 | 48 | OUT5_EN | GPIO | out |  |
| PE10 | 63 | OUT_FAULT_N (commun) | GPIO | in | EXTI10 |

## Bus d'extension

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PE2 | 1 | EXT_SPI_SCK | SPI4_SCK | out |  |
| PE5 | 4 | EXT_SPI_MISO | SPI4_MISO | in |  |
| PE6 | 5 | EXT_SPI_MOSI | SPI4_MOSI | out |  |
| PE4 | 3 | EXT_CS1 | GPIO | out |  |
| PE8 | 59 | EXT_CS2 | GPIO | out |  |
| PG7 | 92 | EXT_IRQ1 | GPIO | in | EXTI7 |
| PG11 | 126 | EXT_IRQ2 | GPIO | in | EXTI11 |
| PB8 | 139 | EXT_UART_RX | UART4_RX | in |  |
| PB9 | 140 | EXT_UART_TX | UART4_TX | out |  |
| PD6 | 122 | EXT_I2C_SCL (EEPROM ID) | I2C3_SCL | io |  |
| PD7 | 123 | EXT_I2C_SDA | I2C3_SDA | io |  |
| PA0 | 34 | EXT_SYNC (TIM5 32 bits) | TIM5_CH1 | io |  |
| PG1 | 57 | EXT_NRESET | GPIO | out |  |

## I2C interne

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PF0 | 10 | I2C_SDA (EEPROM, moniteur d'alim) | I2C2_SDA | io |  |
| PF1 | 11 | I2C_SCL | I2C2_SCL | io |  |

## Mesures analogiques (ADC)

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PC0 | 26 | VBAT_SENSE | ADC1_INP10 | ana |  |
| PC1 | 27 | V5_SENSE | ADC1_INP11 | ana |  |
| PC2 | 28 | VSERVO_SENSE | ADC1_INP12 | ana |  |
| PC3 | 29 | IBAT_SENSE (ampli shunt) | ADC1_INP13 | ana |  |
| PF11 | 49 | PRESS1 (capteur pression réservoir) | ADC1_INP2 | ana |  |
| PF12 | 50 | PRESS2 | ADC1_INP6 | ana |  |
| PF13 | 53 | PRESS3 (chambre) | ADC2_INP2 | ana |  |
| PF14 | 54 | PRESS4 | ADC2_INP6 | ana |  |
| PB0 | 46 | TEMP1 (NTC) | ADC1_INP9 | ana |  |
| PB1 | 47 | TEMP2 (NTC) | ADC1_INP5 | ana |  |
| PC4 | 44 | ANA_EXT1 (connecteur d'extension) | ADC1_INP4 | ana |  |
| PC5 | 45 | ANA_EXT2 (connecteur d'extension) | ADC1_INP8 | ana |  |

## Interface

| Broche | Pin | Signal | Fonction | Dir. | EXTI |
|---|---|---|---|---|---|
| PA8 | 100 | BOUTON (utilisateur) | GPIO | in |  |
| PC14 | 8 | LED_R | GPIO | out |  |
| PC15 | 9 | LED_G | GPIO | out |  |
| PA1 | 35 | LED_B | GPIO | out |  |
| PC6 | 96 | BUZZER (PWM) | TIM3_CH1 | out |  |

## Points d'attention pour le schéma

1. **Pas de SWO.** PB3 sert au SCK du SPI3. Pour le printf de debug, utilise Segger RTT par SWD, ou la console USART1 (PA9/PA10).
2. **PA15, PB3 et PB4 sont des broches JTAG au reset.** Le firmware les reconfigure au démarrage. Mets les éventuelles résistances de tirage (pull-up / pull-down) sur ces lignes pour qu'elles soient dans un état sûr avant l'initialisation.
3. **PC13, PC14 et PC15 sont dans le domaine de sauvegarde.** Elles ne fournissent que ~3 mA et sont lentes. PC14/PC15 pilotent les LED rouge et verte : utilise des LED à faible courant (1–2 mA) ou un transistor. Il n'y a donc pas de quartz 32,768 kHz (LSE) : l'horodatage vient du HSE et du PPS du GNSS.
4. **Pas de ligne EXTI** pour SD_DETECT (PG15), ARMED_STATUS (PG10) et le bouton (PA8) : ces entrées sont lues par scrutation (polling).
5. **Sorties de puissance (OUT1–5) et ACT_PWR_EN** : ajoute des pull-down pour que tout reste coupé pendant le reset et le démarrage. Le superviseur doit pouvoir forcer ces sorties à zéro (porte ET matérielle avec son signal d'autorisation).
6. **SUP_KILL_N et OUT_FAULT_N** sont actifs à l'état bas, avec pull-up : un câble débranché déclenche l'arrêt.
7. **Entrées analogiques** : les capteurs de pression 0,5–4,5 V doivent passer par un pont diviseur vers 0–3,3 V, avec un filtre RC et une protection (diodes ou TVS) côté connecteur.
8. **CAN** : les lignes CANx_TERM_EN commandent une terminaison de 120 Ω commutable (switch analogique ou MOSFET), utile quand Sweeton est en bout de bus.
9. **IMU_HEATER** (PC7, PWM) : chauffe douce de l'IMU pour stabiliser sa température, comme sur les contrôleurs de vol PX4. Tu peux laisser ce transistor non monté au début.
10. **VDDIO2 (pin 121)** : sur les STM32H5, cette alimentation sert une partie du port G (PG2 à PG15, à confirmer dans la datasheet). Beaucoup de signaux de ce brochage sont sur le port G. Relie VDDIO2 au 3,3 V avec son condensateur de découplage, sinon ces broches ne fonctionneront pas.
