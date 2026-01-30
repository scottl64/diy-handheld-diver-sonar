# Complete DIY Handheld Diver Sonar: Bill of Materials and Build Guide

A 200 kHz ultrasonic sonar device rated to 50+ meters can be built for approximately **$450-650** using commercially available components. The **Open Echo** project provides the most proven open-source foundation, while Blue Robotics enclosures solve the critical waterproofing challenge. This guide provides every component needed with specific part numbers, prices, and purchase links.

---

## Core system architecture at a glance

The device consists of six subsystems: a **200 kHz piezoelectric transducer** converts electrical pulses to acoustic waves and receives echoes; the **TUSS4470 driver IC** generates high-voltage transmit bursts and amplifies weak return signals; an **STM32F411 or Teensy 4.1 microcontroller** performs real-time FFT processing; a **10-segment LED bar graph plus 0.96" OLED** displays range information; a **vibration motor with DRV2605L driver** provides tactile feedback; and **18650 lithium cells** power everything inside a **Blue Robotics 3" aluminum enclosure** rated to 900 meters.

The transmit chain generates **100-250V peak-to-peak pulses** at 200 kHz through a step-up transformer, achieving **20-30 meter detection range** in clear water. Echo signals pass through a bandpass filter centered at 200 kHz, then a variable-gain amplifier before digitization at **1+ MSPS**. The microcontroller calculates time-of-flight, applies digital filtering, and updates displays at approximately 10 Hz.

---

## Transducer options and recommendations

The transducer selection determines maximum range, beam width, and build complexity. Three approaches exist at different price points.

### Best value: Steminc SMATR200H19XDA

| Specification | Value |
|--------------|-------|
| **Part Number** | SMATR200H19XDA |
| **Price** | $47.52 + $12.99 shipping |
| **Purchase URL** | https://www.steminc.com/PZT/fr/air-transducer-200-khz |
| **Frequency** | 200 kHz ±4% |
| **Beam Angle** | 14° ±2° (-3dB) |
| **Capacitance** | 220pF ±50pF |
| **Dimensions** | 19mm diameter × 16mm length |
| **Max Drive Voltage** | 500Vpp (2% duty cycle) |
| **Waterproof** | Yes (plastic case with epoxy window) |

This transducer operates as both transmitter and receiver, simplifying the design. The narrow **14° beam** provides good target resolution at the cost of requiring more precise aiming. Stock availability is excellent with 264 units currently listed.

### Budget alternative: eBay 10mm paired transducers

| Specification | Value |
|--------------|-------|
| **Product** | 10mm 200kHz Waterproof Ultrasonic Sensor R+T |
| **Price** | ~$25-30 + $10.99 shipping |
| **Purchase URL** | https://www.ebay.com/itm/290723531325 |
| **Configuration** | Separate TX and RX elements |
| **Frequency** | 200 kHz |

Using separate transmit and receive elements allows simultaneous transmit/receive operation, eliminating the ring-down recovery period that limits minimum detectable range. Over 110 units have sold, indicating reasonable quality for the price.

### Premium option: Blue Robotics Ping Sonar

| Specification | Value |
|--------------|-------|
| **Part Number** | BR-101684 |
| **Price** | $430.00 |
| **Purchase URL** | https://bluerobotics.com/store/sonars/echosounders/ping-sonar-r2-rp/ |
| **Frequency** | 115 kHz (not 200 kHz) |
| **Range** | 100 meters |
| **Depth Rating** | 300 meters |

While operating at 115 kHz rather than 200 kHz, the Ping Sonar is a complete solution with open-source software, Arduino/Python libraries, and proven underwater reliability. It eliminates all analog front-end complexity but costs nearly as much as building the entire DIY device.

**Quantity needed:** 1 transducer (or 1 pair if using separate TX/RX)

---

## Ultrasonic driver electronics

The driver circuit must generate high-voltage pulses to excite the piezoelectric transducer and amplify the microvolt-level return echoes.

### Primary recommendation: Texas Instruments TUSS4470

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **TUSS4470 IC** | TUSS4470TRTJR | $3.40 | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/TUSS4470TRTJR/11502045) |
| **Evaluation Module** | BOOSTXL-TUSS4470 | $298.80 | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/BOOSTXL-TUSS4470/12174715) |

The TUSS4470 operates from **40 kHz to 1 MHz**, perfectly covering the 200 kHz target. It integrates an H-bridge driver capable of 26Vpp direct drive, a logarithmic amplifier receive chain, and SPI control—all in a 4mm × 4mm package. For higher output voltage, use pre-driver mode with external MOSFETs and a step-up transformer.

**Key specifications:**
- Supply voltage: 3.0V to 5.5V (logic), up to 13V (driver)
- Integrated H-bridge for direct transducer drive
- Pre-driver mode outputs for external high-voltage stage
- Logarithmic amplifier for wide dynamic range reception

### Step-up transformer for high-voltage drive

| Component | Part Number | Ratio | Price | Source |
|-----------|------------|-------|-------|--------|
| **TDK Ultrasonic Transformer** | B78416A2386A003 | 1:1:9 | $4.11 | [DigiKey](https://www.digikey.com/en/products/detail/tdk-electronics-inc/B78416A2386A003/9863808) |

This push-pull primary transformer steps up the TUSS4470's output to approximately **100-200Vpp** with external MOSFETs. For 200 kHz operation (above the transformer's 125 kHz spec), consider winding a custom transformer on a 3C90 ferrite core with 1:10 turns ratio.

### Budget alternative: Discrete H-bridge

| Component | Part Number | Price | Qty | Source |
|-----------|------------|-------|-----|--------|
| **TC4427ACPA** | TC4427ACPA | $1.38 | 2 | [DigiKey](https://www.digikey.com/en/products/detail/microchip-technology/TC4427ACPA/115309) |
| **IR2110PBF** | IR2110PBF | $3.24 | 1 | [DigiKey](https://www.digikey.com/en/products/detail/infineon-technologies/IR2110-2PBF/811587) |
| **IRF740 MOSFETs** | IRF740PBF | ~$3.50 | 4 | DigiKey/Mouser |

This approach requires more design effort but offers full control over timing and voltage levels. The TC4427 provides 1.5A gate drive, while the IR2110 enables high-side/low-side control with bootstrap supply up to 500V offset.

---

## Microcontroller selection

Signal processing demands drive microcontroller selection. At 200 kHz, Nyquist requires sampling above 400 kSPS, but practical implementations need **1+ MSPS** with sufficient RAM for FFT buffers.

### Best overall: Teensy 4.1

| Specification | Value |
|--------------|-------|
| **Processor** | ARM Cortex-M7 @ 600 MHz |
| **RAM** | 1024 KB (512K tightly coupled) |
| **Flash** | 8 MB |
| **ADC** | 2× 12-bit, ~500 kSPS internal |
| **Price** | $29.95 |
| **Purchase URL** | [Adafruit](https://www.adafruit.com/product/4622) or [SparkFun](https://www.sparkfun.com/teensy-4-1.html) |

The Teensy 4.1's **600 MHz clock** provides 10× the processing power of typical microcontrollers, enabling complex matched filtering and FFT without optimization struggles. Arduino IDE compatibility eases development. **Note:** Adafruit indicates Teensy boards may be discontinued—purchase while available.

### Best value: STM32F411 Black Pill

| Specification | Value |
|--------------|-------|
| **Processor** | ARM Cortex-M4F @ 100 MHz |
| **RAM** | 128 KB SRAM |
| **Flash** | 512 KB |
| **ADC** | 12-bit, up to 2.4 MSPS |
| **Price** | $4-30 depending on source |
| **Purchase URLs** | [Adafruit](https://www.adafruit.com/product/4877) ($29.95), [AliExpress](https://aliexpress.com) WeAct boards ($4-6) |

The STM32F411's **2.4 MSPS ADC** exceeds requirements, while the **FPU** accelerates floating-point FFT calculations. The CMSIS-DSP library provides optimized signal processing functions. Open Echo uses the simpler STM32F103, validating STM32 platform suitability.

### Budget option: STM32F103C8T6 Blue Pill

| Specification | Value |
|--------------|-------|
| **Processor** | ARM Cortex-M3 @ 72 MHz |
| **RAM** | 20 KB SRAM |
| **ADC** | 12-bit, 1 MSPS |
| **Price** | $2-3 |
| **Purchase URL** | [Amazon 2-pack](https://www.amazon.com/initeq-STM32F103C8T6-Minimum-Development-Programmer/dp/B079B95L9Y) (~$12-15 with programmer) |

The Open Echo project proves this chip works for 200 kHz sonar. Limited RAM restricts FFT size, but 20 KB accommodates 1024-point transforms. Clone quality varies—purchase from reputable sellers.

### External high-speed ADC (if needed)

| Component | Part Number | Sample Rate | Price | Source |
|-----------|------------|-------------|-------|--------|
| **ADS7883** | ADS7883SDBVT | 3 MSPS | $2-5 | [DigiKey](https://www.digikey.com/en/products/base-product/texas-instruments/296/ADS7883/64325) |

If internal ADC proves insufficient, the ADS7883 provides **3 MSPS** 12-bit sampling over SPI. A PMOD breakout design exists on GitHub: https://github.com/mattvenn/ADS7883-pmod

---

## Display and feedback components

Visual and tactile feedback enables underwater operation where audio cues are impractical.

### LED bar graph display

| Component | Part Number | Price | Qty | Source |
|-----------|------------|-------|-----|--------|
| **Kingbright DC10GWA (Green)** | DC10GWA | $4.05 | 2 | [DigiKey](https://www.digikey.com/en/products/detail/kingbright/DC10GWA/1747576) |
| **Kingbright DC10EWA (Red)** | DC10EWA | $3.73 | 2 | [DigiKey](https://www.digikey.com/en/products/detail/kingbright/DC10EWA/1747575) |
| **SparkFun 10-Segment (Blue)** | COM-09937 | $2.95 | 2 | [SparkFun](https://www.sparkfun.com/10-segment-led-bar-graph-blue.html) |

Dual bar graphs—red for near targets, green for far—provide intuitive range indication visible through the polycarbonate dome.

### OLED display

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **Adafruit 0.96" OLED (SSD1306)** | ID 326 | $17.50 | [Adafruit](https://www.adafruit.com/product/326) |
| **Budget 0.96" OLED** | DIYmall SSD1306 | $6-8 | [Amazon](https://www.amazon.com/DIYmall-Serial-128x64-Display-Arduino/dp/B00O2KDQBE) |
| **1.3" OLED (SH1106)** | Hosyond 5-pack | $14 | [Amazon](https://www.amazon.com/Hosyond-Display-Compatible-Arduino-Raspberry/dp/B0C3L7N917) |

The 128×64 OLED displays numeric range, signal strength, battery level, and optional waveform visualization. I2C interface minimizes wiring. Mount behind the polycarbonate dome with optical bonding for underwater visibility.

### Vibration motor and haptic driver

| Component | Part Number | Price | Qty | Source |
|-----------|------------|-------|-----|--------|
| **Vibration Motor Disc** | ADA1201 | $1.95 | 4 | [Adafruit](https://www.adafruit.com/product/1201) |
| **DRV2605L Haptic Driver** | ID 2305 | $7.95 | 1 | [Adafruit](https://www.adafruit.com/product/2305) |

The 10mm × 2.7mm disc motor draws 60-100mA and provides clear tactile feedback. The DRV2605L enables **123 built-in haptic effects** and proportional intensity control via I2C—ping rate can increase as targets approach, providing intuitive proximity sensing without visual attention.

### LED driver IC

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **LM3914N-1** | LM3914N-1/NOPB | $2.25-4.00 | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/LM3914N-1-NOPB/148697) |

The LM3914 converts analog voltage directly to a 10-LED bar or dot display without microcontroller intervention—useful for a dedicated signal strength indicator independent of digital processing.

---

## Power system design

Operating at 50+ meters eliminates surface charging, requiring sufficient capacity for a full dive plus margin.

### Battery cells

| Component | Specification | Price | Source |
|-----------|--------------|-------|--------|
| **Samsung 35E Protected** | 3500mAh, 3.6V, 8A continuous | $11.99 | [18650 Battery Store](https://www.18650batterystore.com/products/samsung-35e-protected) |

Protected cells include overcharge, overdischarge, and short-circuit protection. Two cells in series (7.4V, 3500mAh ≈ **26 Wh**) provide 4+ hours of operation at typical 5W average consumption. The protection circuit adds ~4mm length—verify holder compatibility.

### Battery holders and BMS

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **Dual 18650 Holder** | Keystone 1047 | $3.50 | [Mouser](https://www.mouser.com/ProductDetail/Keystone-Electronics/1047) |
| **2S 8A BMS Board** | JZK 2S-8A | $2.00 (5-pack $9.99) | [Amazon](https://www.amazon.com/JZK-Lithium-Battery-Protection-Discharge/dp/B09VGH38Y5) |
| **TP4056 USB Charger** | TP4056 Type-C | $0.70 (10-pack $6.99) | [Amazon](https://www.amazon.com/HiLetgo-Lithium-Charging-Protection-Functions/dp/B07PKND8KG) |

The 2S BMS handles cell balancing and protection for the series configuration. Charging voltage is 8.4V at 1A maximum.

### Voltage regulators

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **MT3608 Boost Module** | MT3608 | $0.80 (10-pack $7.99) | [Amazon](https://www.amazon.com/MT3608-Converter-Adjustable-Voltage-Regulator/dp/B0BGLGL9RV) |
| **AMS1117-3.3V LDO** | AMS1117-3.3 | $0.70 (10-pack $6.99) | [Amazon](https://www.amazon.com/HiLetgo-AMS1117-3-3-Step-Down-Module-AMS1117-3-3V/dp/B01HXU1NQY) |

From 7.4V battery, the AMS1117 provides 3.3V for the microcontroller while the MT3608 can boost to 12V+ for the TUSS4470 driver section.

### High-voltage generation for transducer

| Component | Description | Price | Source |
|-----------|------------|-------|--------|
| **NOYITO 100V Boost** | 60-97V output, 100W | $15.99 | [Amazon](https://www.amazon.com/NOYITO-Adjustable-Module-10-32V-60-97V/dp/B07BGT4DYS) |
| **DRV2700** | Integrated 105V boost + piezo driver | $3-5 | [DigiKey](https://www.digikey.com/en/product-highlight/t/texas-instruments/drv2700-high-voltage-driver) |

Typical piezo transducers need 50-200Vpp drive for adequate acoustic output. The DRV2700 integrates boost conversion and differential amplifier specifically for piezo driving.

### Waterproof switch

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **Blue Robotics Switch (1000m)** | BR-100433 | $28.00 | [Blue Robotics](https://bluerobotics.com/store/comm-control-power/switch/switch-10-5a-r1/) |

This is the **only COTS switch verified for deep underwater use**. Standard IP68 switches are rated for 1 meter, not 50 meters. The Blue Robotics switch uses M10×1.5 threads compatible with their end cap penetrator holes.

---

## Analog front end for echo reception

Received echoes may be only microvolts—requiring **80-100 dB** of gain with low noise to achieve 20-30 meter range.

### Preamplifier stage

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **AD620 Instrumentation Amp** | AD620ANZ | $8-12 | [DigiKey](https://www.digikey.com/en/product-highlight/a/analog-devices/ad620-instrumentation-amplifier) |
| **TL072 Dual Op-Amp** | TL072CP | $0.50-0.80 | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/TL072CP/277421) |
| **LMH6643 High-Speed Op-Amp** | LMH6643MA | $2.50-3.50 | [Mouser](https://www.mouser.com/ProductDetail/Texas-Instruments/LMH6643MA-NOPB) |

The AD620 provides **up to 10,000× gain** set by a single resistor, with 9 nV/√Hz input noise—excellent for the first amplification stage directly connected to the transducer. Follow with TL072 stages for additional gain and filtering.

### Variable gain amplifier

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **AD603 VGA** | AD603ARZ | $15-20 | [DigiKey](https://www.digikey.com/en/products/base-product/analog-devices-inc/505/AD603/24475) |
| **VCA810** | VCA810ID | $8-12 | [TI.com](https://www.ti.com/product/VCA810) |

Time-varying gain (TVG) compensates for propagation losses—distant targets need more amplification than near ones. The AD603 provides **40 dB range** per stage; cascade two for 80 dB total.

### Bandpass filter components (200 kHz)

For a second-order LC bandpass centered at 200 kHz with Q=10:

| Component | Value | Price | Source |
|-----------|-------|-------|--------|
| **Inductor** | 820 µH | $0.80 | Coilcraft/TDK via DigiKey |
| **Capacitor (C0G)** | 820 pF | $0.10 | Murata GRM series via DigiKey |
| **Resistor (1%)** | 500Ω | $0.05 | Any metal film |

The LC bandpass rejects out-of-band noise before amplification. Use C0G/NP0 ceramic capacitors for temperature stability.

### Protection diodes

| Component | Part Number | Price | Qty | Source |
|-----------|------------|-------|-----|--------|
| **1N4148** | 1N4148 | $0.10 | 20 | [DigiKey](https://www.digikey.com/en/products/detail/onsemi/1N4148/458603) |
| **BAV99** | BAV99 | $0.10 | 6 | [DigiKey](https://www.digikey.com/en/products/base-product/diodes-incorporated/31/BAV99/1437) |
| **SMBJ5.0A TVS** | SMBJ5.0A | $0.25 | 4 | [DigiKey](https://www.digikey.com/en/products/detail/littelfuse-inc/SMBJ5-0A/285950) |

Back-to-back diodes clamp the input during transmit pulses, protecting sensitive amplifier inputs. TVS diodes provide secondary protection against transients.

### Integrated AFE option

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **TDC1000** | TDC1000PW | $6-7 | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/TDC1000PW/5172356) |

The TDC1000 integrates **LNA, PGA, TX driver, and comparators** specifically for ultrasonic time-of-flight measurement from 31.25 kHz to 4 MHz. Combined with the TUSS4470, it forms a near-complete analog signal chain requiring minimal external components.

---

## Waterproof enclosure system

The housing must withstand **6 bar pressure** at 50 meters (50m × 0.1 bar/m + 1 atm) with generous safety margin.

### Blue Robotics 3" series enclosure

| Component | Part Number | Depth Rating | Price | Source |
|-----------|------------|--------------|-------|--------|
| **3" Aluminum Tube 300mm** | BR-100611-300 | 450-900m* | ~$61 base | [Blue Robotics](https://bluerobotics.com/store/watertight-enclosures/wte-vp/) |
| **3" O-Ring Flange** | BR-100647 | Included | Included | Included |
| **3" Aluminum End Cap (4×M10)** | BR-100949-004 | 1000m | Included | Included |
| **3" Polycarbonate Dome** | BR-101059 | 750m | ~$40-60 | [Blue Robotics](https://bluerobotics.com/store/watertight-enclosures/locking-series/wte-end-cap-vp/) |

The 300mm tube length accommodates the 280mm device specification. Internal diameter is 79mm (aluminum) or 76.2mm (acrylic), providing ample space for electronics. *Current aluminum tubes have temporary derating—verify current specifications before purchase.

### Cable penetrators

| Component | Part Number | Price | Source |
|-----------|------------|-------|--------|
| **WetLink Penetrator 6.5mm** | BR-100870-065 | $13-17 | [Blue Robotics](https://bluerobotics.com/store/cables-connectors/penetrators/wlp-vp/) |
| **WetLink Blank** | WL-BLANK-VP | $5-8 | Blue Robotics |
| **WetLink Plug Wrench** | — | $15-20 | Blue Robotics |

WetLink penetrators seal cables passing through the end cap using compression fittings rated to **1000 meters**. Use blanks to seal unused holes.

### Sealing compounds and lubricant

| Component | Description | Price | Source |
|-----------|------------|-------|--------|
| **Molykote 111** | Silicone O-ring lubricant | $2.99-5.00 (6g) | [Blue Robotics](https://bluerobotics.com/store/watertight-enclosures/enclosure-tools-supplies/molykote/) |
| **West System 105-A + 206-A** | Marine epoxy (Resin + Slow Hardener) | $60-80 | [Amazon](https://www.amazon.com/west-system-105-epoxy-resin/s?k=west+system+105+epoxy+resin) |
| **3M Marine 5200** | Permanent underwater sealant | $12-18 | Amazon |

Molykote 111 lubricates O-rings without degrading Buna-N rubber. West System epoxy potting protects electronics from moisture ingress and provides mechanical support.

### O-rings

| Series | Face Seal | Radial Seal | Price |
|--------|----------|-------------|-------|
| 3" Series | -148 Buna-N 70A | -150 Buna-N 70A | ~$5/set |

Purchase spare O-ring sets from Blue Robotics. Inspect before every dive—a single hair can cause catastrophic flooding.

---

## Reference projects and firmware

### Open Echo: The definitive open-source sonar

| Resource | URL |
|----------|-----|
| **GitHub Repository** | https://github.com/Neumi/open_echo |
| **Hackaday.io Project** | https://hackaday.io/project/196793-a-diy-open-source-sonar |
| **Discord Community** | https://discord.com/invite/rerCyqAcrw |
| **YouTube Channel** | https://www.youtube.com/neumi |

Open Echo uses the TUSS4470 on an Arduino shield, supports transducers from 40 kHz to 1 MHz, outputs NMEA0183 depth sentences, and includes Python visualization software. The creator sells assembled boards for €50 plus shipping—a valuable reference even if building custom.

**Key BOM items from Open Echo:**
- TUSS4470 Arduino Shield
- 200kHz PZT transducers from AliExpress (~€10)
- MT3608 boost converter for USB to 15-20V
- Step-up transformer for 250Vpp drive

### Signal processing libraries

| Library | Platform | URL |
|---------|----------|-----|
| **ArduinoFFT** | Arduino/Teensy | https://github.com/kosme/arduinoFFT |
| **CMSIS-DSP** | STM32 (ARM) | https://www.st.com/en/embedded-software/x-cube-dspdemo.html |
| **U8g2** | Universal displays | https://github.com/olikraus/u8g2 |
| **Adafruit SSD1306** | OLED displays | https://github.com/adafruit/Adafruit_SSD1306 |
| **NewPing** | Ultrasonic sensors | https://docs.arduino.cc/libraries/newping/ |

For STM32, the CMSIS-DSP library provides optimized `arm_rfft_fast_f32()` requiring 50% less computation than complex FFT. Phil's Lab #111 on YouTube demonstrates STM32 FFT implementation step-by-step.

---

## Complete bill of materials summary

### Electronics (~$180-250)

| Category | Component | Qty | Unit Price | Subtotal |
|----------|-----------|-----|------------|----------|
| Transducer | Steminc SMATR200H19XDA | 1 | $60 | $60 |
| Driver IC | TUSS4470TRTJR | 1 | $3.40 | $3.40 |
| Transformer | B78416A2386A003 | 1 | $4.11 | $4.11 |
| MOSFETs | IRFH5020 (or similar) | 4 | $2.00 | $8.00 |
| MCU | Teensy 4.1 | 1 | $29.95 | $29.95 |
| AFE IC | TDC1000PW | 1 | $7.00 | $7.00 |
| Preamp | AD620ANZ | 1 | $10.00 | $10.00 |
| Op-amps | TL072CP | 3 | $0.60 | $1.80 |
| VGA | AD603ARZ | 1 | $17.00 | $17.00 |
| OLED Display | Adafruit 326 | 1 | $17.50 | $17.50 |
| LED Bar Graph | DC10GWA | 2 | $4.05 | $8.10 |
| Bar Driver | LM3914N-1 | 2 | $3.00 | $6.00 |
| Vibration Motor | ADA1201 | 2 | $1.95 | $3.90 |
| Haptic Driver | DRV2605L | 1 | $7.95 | $7.95 |
| Protection | Diodes, TVS, caps | — | — | $10.00 |
| Passives | Resistors, caps, inductors | — | — | $15.00 |
| **Electronics Subtotal** | | | | **$209.71** |

### Power system (~$75-90)

| Component | Qty | Unit Price | Subtotal |
|-----------|-----|------------|----------|
| Samsung 35E Protected 18650 | 2 | $11.99 | $23.98 |
| Dual Battery Holder | 1 | $3.50 | $3.50 |
| 2S BMS Board | 1 | $2.00 | $2.00 |
| TP4056 Charger | 1 | $0.70 | $0.70 |
| MT3608 5V Boost | 2 | $0.80 | $1.60 |
| AMS1117-3.3V LDO | 2 | $0.70 | $1.40 |
| Blue Robotics Switch | 1 | $28.00 | $28.00 |
| NOYITO HV Boost | 1 | $15.99 | $15.99 |
| **Power Subtotal** | | | **$77.17** |

### Enclosure and waterproofing (~$200-250)

| Component | Qty | Unit Price | Subtotal |
|-----------|-----|------------|----------|
| 3" Aluminum Enclosure 300mm | 1 | $61.00 | $61.00 |
| 3" Polycarbonate Dome | 1 | $50.00 | $50.00 |
| WetLink Penetrators | 2 | $15.00 | $30.00 |
| WetLink Blanks | 2 | $6.00 | $12.00 |
| Molykote 111 | 1 | $5.00 | $5.00 |
| West System Epoxy Kit | 1 | $70.00 | $70.00 |
| Spare O-Rings | 1 | $5.00 | $5.00 |
| **Enclosure Subtotal** | | | **$233.00** |

### Tools and testing (~$100-400)

| Tool | Price Range | Notes |
|------|-------------|-------|
| Oscilloscope (Rigol DS1054Z) | $350-400 | Essential for debugging |
| Soldering Station (Hakko FX-888D) | $100-120 | Temperature-controlled |
| Multimeter (True RMS) | $30-100 | Capacitance measurement needed |
| Hot Air Rework | $50-80 | For SMD components |
| PCB Order (JLCPCB) | $20-50 | 5 boards + shipping |

### Grand total estimate

| Category | Cost Range |
|----------|------------|
| Electronics | $180-250 |
| Power System | $75-90 |
| Enclosure | $200-250 |
| Tools (if needed) | $100-400 |
| Shipping (estimated) | $50-100 |
| **Total without tools** | **$505-690** |
| **Total with tools** | **$605-1090** |

---

## Critical build notes

**Transducer waterproofing:** Even "waterproof" transducers should be potted in marine epoxy where cables enter, as pressure at depth will force water through imperfect seals.

**Vacuum testing is mandatory:** Before any water deployment, pull vacuum on the enclosure and monitor for pressure change over 15 minutes. A slow leak at 0.5 bar vacuum becomes catastrophic flooding at 6 bar positive pressure.

**Signal processing complexity:** Echo detection involves more than simple threshold crossing. Implement time-varying gain to compensate for propagation loss (~0.5 dB/m at 200 kHz in seawater), matched filtering to improve SNR, and envelope detection to extract range from the modulated carrier.

**Acoustic coupling:** The transducer must have direct water contact or be coupled through an acoustically transparent window. Mounting behind thick plastic or inside a potted enclosure will severely attenuate signals.

**Temperature effects:** Sound speed varies ~3 m/s per °C. At 20-30m range, a 5°C temperature error introduces ~0.5m range error—acceptable for obstacle detection but relevant for precision applications.

## Resources & Community

The [Open Echo project](https://github.com/Neumi/open_echo) by Neumi is the most relevant open-source foundation for this build. While the project is primarily designed for boat-mounted bathymetry and fish-finding, the creator has successfully built and tested a working handheld dive SONAR (July 2025) using the Open Echo All-In-One board, Raspberry Pi, and 7" display - achieving 25m+ underwater range in a CNC-machined polyethylene housing.

**Important note:** There isn't a detailed step-by-step dive sonar build guide published yet. The dive sonar details are shared mainly through the Discord community. However, the core electronics, firmware, and principles are well-documented and could be adapted for this Aliens-inspired handheld concept.

**Key resources:**
- [GitHub: Neumi/open_echo](https://github.com/Neumi/open_echo) - Open source schematics, board layouts, and firmware (40kHz-1000kHz support)
- [Hackaday.io project page](https://hackaday.io/project/196793-a-diy-open-source-sonar) - Build logs and development details
- [Dive sonar firmware (STM32F103)](https://gist.github.com/Neumi/07914c59210a4e1291520d336d150dad) - Code used for the working dive sonar
- [Discord community](https://discord.com/invite/rerCyqAcrw) - **Best place for dive sonar specifics**, active support, and to see build photos
- [YouTube: Neumi](https://www.youtube.com/neumi) - Development videos

**Hardware option:** The TUSS4470 Arduino Shield (~$50 from Elecrow) is a good starting point for learning and prototyping before committing to a full dive-rated build.
