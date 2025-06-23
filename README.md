# re-yootech-charger
Reverse Engineering of Yootech Wireless Charger 
![Yootech Wireless Charger ](https://m.media-amazon.com/images/I/61oIAKY9s1L._AC_SL1500_.jpg)
## PCB POI's
- PCB Serial Number: `TX-F500-330-V11`, `2021-11-11`
- **U1**: CVSMicro IC: `CV90330` `2135X`
  - [datasheet](https://static2.xunxiang.site/uploads/sites/547/2022/06/c5802f62f9b19c7a5d5f58fb09c67df9.pdf)
    - [cpu datasheet](https://www.nuvoton.com/resource-files/DS_MS51_8K_Series_EN_Rev1.00.pdf)
- **Q1** and **Q2**: MOSFET `Vs` `3622DE` `871J36`
  - [datasheet](https://www.vgsemi.com/upload/attachment/goods/VS3622DE.pdf)
- Test Pads: `D-`, `D+`, `CUR`, `CODE1`, `CODE2`
- **U8**: _unknown IC_
- **J3**: 4-pin header, looks like programming port / debug?
- **J1**: USB-C power ... and _data_?
- **U2**: Voltage Regulator `6239A` `2135 / 50`
  - [datasheet](https://static.chipdip.ru/lib/727/DOC043727527.pdf)

## CVSMicro IC
![Typical Applications](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRsBlzpf0kj5wMS2Zlhi8juTVe_riwVaPJSYw&s)
- Part number: `CV90330` `2135X`
- 16kB non-volatile memory
- 1T8051 CPU
- QC/SCP/Phy & Controller, connected to `D+` and `D-`
### Pin Descriptions
| Pin No. | Name | Description |
|---------|------|-------------|
| 1       | AVSS | Ground      |
| 2       | AVDD | Power Source |
| 3       |  DP  | USB D+ interface |
| 4       |  DM  | USB D+ interface |
| 5       | VSS  | Ground      |
| 6       | MCLK | Emulation port clock |
| 7       | MDAT | Emulation port data |
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


## Sources
1. [Amazon Prodct Page](https://www.amazon.com/Wireless-Qi-Certified-Charging-Compatible-Qi-Enabled/dp/B079KZ49PJ)
2. [Yootech Product Page](https://yootech.net/services/)
