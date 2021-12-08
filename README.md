# hass-AMS - AMS Reader for Norwegian and swedish AMS meters
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge)](https://github.com/custom-components/hacs)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=AS5PQHAERQ85J&currency_code=NOK&source=url)

Custom component reading AMS through MBus adapter into HomeAssistant
Supports the new energy dashboard in Home-Assistant.

If it does not decode your data, please submit a ticket, and I will try to 
make a parser for your meter.
If your meter type shown "unknown", please submit a ticket, and I will add 
your meter to the module.

Works with the following swedish and norwegian meters:

Kamstrup:
  - 6861111 tested by janna at homeassistant community forum
  - 6841121 tested by me
  - 6841131
  - 6841138A tested by NilsFA at homeassistant community forum
  - 6851121
  - 6851131
 
Kaifa:
Norway: 
  - MA304H3E Thanks to @thomasja27 for testing :+1:

Sweden:
  - MA304H4 Thanks to @runlar for testing (swedish version) :+1:

Not tested with, but should work:
  - MA105H2E
  - MA304T4
  - MA304T3

Aidon:
Norway:
 - 6525 Thanks to @razzymoose for testing and providing patch :+1:
 - 6515 Thanks to @maxgyver87 for fault finding and testing :+1:
 - 6534 Thanks to @mariwing for testing and debugging :+1:
 - 6483 Thanks @PerBob81 for confirming :+1:

Sweden:
 - 6484 Thanks to @bo1jo for testing and debugging :+1:


Not tested with, but should work:
 
 Norway:
 - 6540
 - 6550
 
 Sweden:
 - 6479
 
## *Installation
Easiest method is to install via HACS. Then setup via *Integrations* config.
*Or*
1. Copy *ams* folder into your *custom_components* folder.
2. Config by YAML setup or config by integrations in Home-assistant

*YAML options*
```yaml
ams:
  serial_port: '/dev/ttyUSB0' # Required. The serial port used to communicate through
  baudrate: 2400 # Optional, defaults to '2400'
  parity: 'N'  # Optional, defaults to 'N'
  meter_manufacturer: 'auto' # Optional, defaults to 'auto'
```

  
Start Home-Assistant, 
Set up the integration in the *Integrations* config if you haven't set up by YAML config.

For parity options see https://github.com/pyserial/pyserial/blob/master/serial/serialutil.py#L79

Meter manufacturer field options are:
```
'auto'
'aidon'
'aidon_se' # Swedish aidon meter RF2 modules
'kamstrup'
'kaifa'
'kaifa_se' # Swedish kaifa meters
```
This will create sensors for each of the available usage data in the meter.
The accumulative sensors will only be fully available after first read, and is transmitted from the meter 5 seconds past the hour.
There seems to be a bug in the current Kamstrup firmware that the hour package is transmitted at xx:xx:55.

## *Known working modules*
https://www.aliexpress.com/item/32719562958.html?spm=a2g0s.9042311.0.0.c8314c4dpbv1pv
https://www.aliexpress.com/item/32751482255.html?spm=2114.10010108.1000014.1.2a3189f8fCOsSM
https://www.aliexpress.com/item/32834331647.html?spm=a2g0o.detail.1000060.1.74cfdcd4qts4jp

## *Known NOT working modules*
https://www.aliexpress.com/item/32814808312.html?shortkey=iM7rQb67&addresstype=600

Improvements and suggestions are also welcome.
Keep in mind, I am not a experienced programmer :)
Enjoy

Latest information about OBIS for all the norwegian meters: https://www.nek.no/info-ams-han-utviklere/

Latest information about swedish standard for AMS: https://www.energiforetagen.se/globalassets/energiforetagen/det-erbjuder-vi/kurser-och-konferenser/elnat/branschrekommendation-lokalt-granssnitt-v2_0-201912.pdf

Content of the raw data packages (Kamstrup example, collected info from 
various web sites and documents):
```
Header:

7E          <-- Frame start flag
A           <-- 4 bits, A = 0b1010 = frame format type 3
0E2         <-- 1 bit, segmentation bit + 11 bits, frame length sub-field, 0xE2 = 226 bytes (excluding opening and closing frame flags)
2B          <-- Destination address, 1 bit, 0b1 = unicast + 6 bit, node address, 0b010101 = 21 + 1 bit, address size, 0b1 = 1 byte
21          <-- Source address, 1 bit, 0b1 = unicast + 6bit, node address, 0b010000 = 16 + 1 bit, address size, 0b1 = 1 byte
13          <-- Control field
23 9A       <-- Header check sequence (HCS) field, CRC-16/X-25 (Reverse order)


Information:

E6                  <-- Destination LSAP
E7                  <-- Source LSAP, LSB = 0b1 = command
00                  <-- LLC Quality
0F                  <-- LLC Service Data Unit
00 00 00 00         <-- "Long-Invoke-Id-And-Priority"?
0C                  <-- string length?, 0C = 12
   07 E3            <-- Full year, 0x07E3 = 2019
   06               <-- Month, June
   13               <-- Day of month, 0x12 = 19
   02               <-- Day of week, Friday
   15               <-- Hour of day, 0x14 = 21
   2E               <-- Minute of hour, 0x2F = 46
   32               <-- Second of minute, 0x32 = 50
   FF               <-- Hundredths of second, 0xFF = not specified
   80 00            <-- Deviation (offset from UTC), 0x8000 = not specified
   80               <-- Clock status, 0x80 = 0b10000000, MSB 1 = summer time
   
02                                                        <-- struct
   19                                                     <-- 0x19 = 25 elements

0A                                                        <-- visible-string
   0E                                                     <-- string length 0x0E = 14 bytes
       4B 61 6D 73 74 72 75 70 5F 56 30 30 30 31          <-- OBIS List Version Identifier, Kamstrup_V0001

09                                                        <-- octet-string
   06                                                     <-- string length, 0x06 = 6 bytes
      01 01 00 00 05 FF                                   <-- OBIS for Meter ID, 1.1.0.0.5.255
0A                                                        <-- visible-string
   10                                                     <-- string length, 10 = 16 bytes  
       31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36    <-- Meter ID, fake

09                                                        <-- octet-string
   06                                                     <-- string length, 0x06 = 6 bytes
      01 01 60 01 01 FF                                   <-- OBIS for meter type, 1.1.96.1.1.255
0A                                                        <-- visible-string
   12                                                     <-- string lenth, 0x12 = 18 bytes
   36 38 34 31 31 33 31 42 4E 32 34 33 31 30 31 30 34 30  <-- Meter type, 6841131BN243101040)

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 01 07 00 FF     <-- OBIS for Active Power +, 1.1.1.7.0.255
06                          <-- unsigned, 4 bytes
   00 00 06 A7              <-- 0x06A7 = 1703 Watts

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 02 07 00 FF     <-- OBIS for Active Power -, 1.1.2.7.0.255
06                          <-- unsigned, 4 bytes
    00 00 00 00             <-- 0 Watt
    
09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 03 07 00 FF     <-- OBIS for Reactive Power +, 1.1.3.7.0.255
06                          <-- unsigned, 4 bytes
    00 00 00 00             <-- 0 Watt

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 04 07 00 FF     <-- OBIS for Reactive Power -, 1.1.4.7.0.255
06                          <-- unsigned, 4 bytes
   00 00 01 E0              <-- 0x01E0 = 480 Watts

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 1F 07 00 FF     <-- OBIS for L1 Current, 1.1.31.7.0.255
06                          <-- unsigned, 4 bytes
   00 00 00 88              <-- 1.36 Ampere

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 33 07 00 FF     <-- OBIS for L2 Current, 1.1.51.7.0.255
06                          <-- unsigned, 4 bytes
   00 00 02 36              <-- 5.66 Ampere

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 47 07 00 FF     <-- OBIS for L3 Current, 1.1.71.7.0.255
06                          <-- unsigned, 4 bytes
   00 00 00 6D              <-- 1.09 Ampere
   
09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 20 07 00 FF     <-- OBIS for L1 Voltage, 1.1.32.7.0.255
12                          <-- unsigned, 2 bytes
   00 EB                    <-- 235 Volt

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 34 07 00 FF     <-- OBIS for L2 Voltage, 1.1.52.7.0.255
12                          <-- unsigned, 2 bytes
   00 EB                    <-- 235 Volt

09                          <-- octet-string
   06                       <-- string length, 0x06 = 6 bytes
      01 01 48 07 00 FF     <-- OBIS for L3 Voltage, 1.1.72.7.0.255
12                          <-- unsigned, 2 bytes
   00 EB                    <-- 235 Volt


End:

83 77                       <-- Frame check sequence (FCS) field, 
                                CRC-16/X-25, (invalid) Reversed
7E                          <-- Frame end flag
```