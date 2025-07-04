# re-yootech-charger
Reverse Engineering of Yootech Wireless Charger FCC ID: [2AOSUF500-330](https://fccid.io/2AOSUF500-330)
![Yootech Wireless Charger ](https://m.media-amazon.com/images/I/61oIAKY9s1L._AC_SL1500_.jpg)
## PCB POI's
- PCB Serial Number: `TX-F500-330-V11`, `2021-11-11`
- **U1**: CVSMicro IC: `CV90330` `2135X`
  - [datasheet](https://static2.xunxiang.site/uploads/sites/547/2022/06/c5802f62f9b19c7a5d5f58fb09c67df9.pdf)
    - [cpu datasheet](https://www.nuvoton.com/resource-files/DS_MS51_8K_Series_EN_Rev1.00.pdf)
  - [Product Description](http://www.chipsvision.com.cn/en/product/82.html)
- **Q1** and **Q2**: MOSFET `Vs` `3622DE` `871J36`
  - [datasheet](https://www.vgsemi.com/upload/attachment/goods/VS3622DE.pdf)
- Test Pads:
  - USB: `D-`, `D+`
  - U1: `CUR`, `CODE1`, `CODE2`
- **U8**: _unknown IC_
- **J3**: 4-pin header, looks like programming port / debug?
  - Likely connects to U1 MCLK/MDAT
- **J1**: USB-C power ... and _data_?
- **U2**: Voltage Regulator `6239A` `2135 / 50`
  - [datasheet](https://static.chipdip.ru/lib/727/DOC043727527.pdf)

## CVSMicro IC
![Typical Applications](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRsBlzpf0kj5wMS2Zlhi8juTVe_riwVaPJSYw&s)
- Part number: `CV90330` `2135X`
- 16kB non-volatile memory
- 1T8051 CPU
- QC/SCP/Phy & Controller, connected to `D+` and `D-`
- MCLK/MDAT connected to 4-pin header "programming port"
- [CV90331 Datasheet](https://static2.xunxiang.site/uploads/sites/547/2022/06/c5802f62f9b19c7a5d5f58fb09c67df9.pdf)
- [jlpcb product page](https://jlcpcb.com/partdetail/-CV90331/C9900001993)
### Pin Descriptions
| Pin No. | Name | Description |
|---------|------|-------------|
| 1       | AVSS | Ground      |
| 2       | AVDD | Power Source |
| 3       |  DP  | USB D+ interface |
| 4       |  DM  | USB D+ interface |
| 5       | VSS  | Ground      |
| **6**       | **MCLK** | **Emulation port clock** |
| **7**       | **MDAT** | **Emulation port data** |
| 8       | P01  | GPIO/External Interrupt |
| 9       | P02  | GPIO/Serial 0 tx data |
| 10      | P03  | GPIO        |
| 11      | P13  | GPIO/Capture 0 |
| 18      | P17  | GPIO/I2C/SDA/ADC |
| 12 - 32 | See Datasheet for rest | |
### CPU 1T8051 (8051)
- [datasheet](https://www.nuvoton.com/resource-files/DS_MS51_8K_Series_EN_Rev1.00.pdf)
- MCS-51 ISA
  - [ISA](https://www.keil.com/dd/docs/datashts/intel/ism51.pdf)
  - [Minimal ISA docs](https://www2.pcs.usp.br/~labdig/manuais/MCS51-InstructionSet.pdf)
  - [User Manual](https://web.mit.edu/6.115/www/document/8051.pdf)
  - [User Manual](https://bitsavers.org/components/intel/8051/MCS-51_Users_Manual_Feb94.pdf)
- Operates up to 24MHz
- Memory
  - 8K Bytes of APROM for User Code
  - 4/3/2/1 Kbytes of Flash for loader (LDROM) configure from APROM for In-System-
Programmable (ISP)
  - Flash Memory accumulated with pages of 128 Bytes from APROM by In-Application-
Programmable (IAP) means whole APROM can be use as Data Flash
  - An additional 128 bytes security protection memory SPROM
  – Code lock for security by CONFIG
  – 256 Bytes on-chip RAM.
  – Additional 1K Bytes on-chip auxiliary RAM (XRAM) accessed by MOVX instruction.
- Potentially useful open source [8051 Emulator](https://github.com/jarikomppa/emu8051)

## Unpowered continuity testing
- Verified **J3** header connections:
  - 0 PWR (?V)
  - 1 GND
  - 2 MDAT to **U1:7**
  - 3 MCLK to **U1:6**
 
## Debugger connection
Will likely need [NU-LINK-PRO/](https://www.digikey.com/en/products/detail/nuvoton-technology-corporation/NU-LINK-PRO/3065247) to connect/debug 8501 within the CV90331. Hardware has been ordered.

## Debugger initial results.
```
| PCB    | Nu-Link-Pro Debugger |

[ ] MCLK -> ICECLK
[ ] MDAT -> ICEDAT
[ ] GND  -> GND
[ ] VCC  -> VDD
```
### Output
Installed `nuvoprog2` and ran command: `~/go/bin/nuvoprog2 -v devices` when connected to PCB
```
[0001:0007:00] 2025/07/03 21:01:35 >  013effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
2025/07/03 21:01:35 <  0118261d000001055500a198536e0b003014000020006100f9ef000018000f44b3e5ebe6c9a0ff6f943944925d7e24cbd1cbf6ebef46ee56807380a821711070
      Nu-Link-Me - Firmware Version 7462 (Target voltage: 0.011000; USB voltage 5.168000)
```
| Section            | Field Name                     | Offset | Length | Raw Value           | Human-Readable Value               |
|--------------------|---------------------------------|--------|--------|----------------------|------------------------------------|
| Header             | Packet Type                    | 0x00   | 1      | `01`                 | Response packet                    |
| Header             | Payload Length?                | 0x01   | 1      | `18`                 | Likely payload size                |
| Header             | Message ID                     | 0x02   | 1      | `26`                 | Message index or opcode            |
| Header             | Response Code                  | 0x03   | 1      | `1d`                 | Status indicator                   |
| Target             | Target ID                      | 0x04   | 4      | `00000105`           | Possibly 0x0105 = MS51 variant?    |
| Control            | Sync Header                    | 0x08   | 2      | `5500`               | Sync marker                        |
| Firmware           | Firmware Version               | 0x0A   | 2      | `a198`               | 0x98A1 = **7462**                  |
| Serial             | Label Marker ("Sn")            | 0x0C   | 2      | `536e`               | ASCII `"Sn"`                       |
| Serial             | Serial Number Length           | 0x0E   | 1      | `0b`                 | 11 bytes                           |
| Serial             | Serial Number                  | 0x0F   | 11     | `003014000020006100f9ef` | Device ID / SN                 |
| Status             | Unknown (reserved or legacy)   | 0x1A   | 4      | `00001800`           | Possibly unused / placeholder      |
| Voltage            | USB Index?                     | 0x1E   | 1      | `0f`                 | Possibly USB port index            |
| Voltage            | USB Voltage (float32 LE)       | 0x1F   | 4      | `44b3e5eb`           | **5.168 V**                        |
| Voltage            | Target Voltage (float32 LE)    | 0x23   | 4      | `e6c9a0ff`           | **0.011 V**                        |
| Signature          | Hash or Device UUID            | 0x27   | 8      | `6f943944925d7e24`   | Unknown, possibly chip signature   |
| Result             | Status/Feature Bitmap          | 0x2F   | 8      | `cbd1cbf6ebef46ee`   | Internal status flags              |
| Footer             | Debugger Config/Temp           | 0x37   | 9      | `56807380a821711070` | Unknown, may include sensor/flags  |


## Sources
1. [Amazon Prodct Page](https://www.amazon.com/Wireless-Qi-Certified-Charging-Compatible-Qi-Enabled/dp/B079KZ49PJ)
2. [Yootech Product Page](https://yootech.net/services/)
3. fccid.io [resources](https://fccid.io/2AOSUF500-330)
