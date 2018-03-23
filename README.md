# Electron Cloud Debug - SD card version

*Special code for debugging cloud connection issues with the Particle Electron to an SD card*

## What is this?

This is a tool to debug cloud connection issues. It:

- Prints out cellular parameters (ICCID, etc.)
- Prints out your cellular carrier and frequency band information
- Pings your IP gateway
- Pings the Google DNS (8.8.8.8)
- Does a DNS server lookup of the device server (device.spark.io)
- Makes an actual cloud connection
- Acts like Tinker after connecting 
- Can print out information about nearby cellular towers

It uses a special debug version of the system firmware, so there's additional debugging information generated as well.

Here's a bit of the output log:

```
service rat=GSM mcc=310, mnc=11094, lac=2 ci=a782 band=GSM 850 bsic=3b arfcn=ed rxlev=40
neighbor 0 rat=GSM mcc=310, mnc=11094, lac=80592df ci=a56f band=GSM 850 bsic=18 arfcn=eb rxlev=37
neighbor 1 rat=GSM mcc=310, mnc=11094, lac=100 ci=a5f2 band=GSM 850 bsic=25 arfcn=b4 rxlev=22
    15.443 AT send      20 "AT+UPING=\"8.8.8.8\"\r\n"
    15.443 AT read  +   14 "\r\n+CIEV: 2,2\r\n"
    15.484 AT read OK    6 "\r\nOK\r\n"
ping addr 8.8.8.8=1
    15.484 AT send      31 "AT+UDNSRN=0,\"device.spark.io\"\r\n"
    17.204 AT read  +   67 "\r\n+UUPING: 1,32,\"google-public-dns-a.google.com\",\"8.8.8.8\",55,812\r\n"
    17.865 AT read  +   67 "\r\n+UUPING: 2,32,\"google-public-dns-a.google.com\",\"8.8.8.8\",55,651\r\n"
    17.936 AT read  +   27 "\r\n+UDNSRN: \"52.91.48.237\"\r\n"
    17.946 AT read OK    6 "\r\nOK\r\n"
```

The source code is [here](https://github.com/rickkas7/electron-clouddebug/blob/master/clouddebug-electron.cpp) so you can see how it works. 

## Prerequisites 

- You should have the [Particle CLI](https://docs.particle.io/guide/tools-and-features/cli/electron/) installed
- You must have a working dfu-util

You'll need a SD card reader, presumably a Micro SD card reader. Make sure you get one that's compatible with the 3.3V logic levels used on the Photon and Electron. Some readers designed for the Arduino expect the 5V logic levels used by the original Arduino. I use [this one from Sparkfun](https://www.sparkfun.com/products/13743) but there are others.

![Electron](electron.jpg)

The SD card reader connects via the SPI interface. There are two SPI interfaces on the Photon and Electron and you can use either. Note that each SPI device must have a separate SS (sometimes called CS) pin. While it's listed in the table below as A2 (or D5), you can use any GPIO pin.

Primary SPI (SPI object)

- A2 SS
- A3 SCK
- A4 MISO
- A5 MOSI

Secondary SPI (SPI1 object)

- D5 SS
- D4 SCK
- D3 MISO
- D2 MOSI

In the picture above:

| Device | SPI Name | SD Reader | Color  |
| ------ | -------- | --------- | ------ |
| A2     | SS       | /CS       | Yellow |
| A3     | SCK      | SCK       | Orange |
| A4     | MISO     | DO        | Blue   |
| A5     | MOSI     | DI        | Green  |
| 3V3    |          | VCC       | Red    |
| GND    |          | GND       | Black  |
|        |          | CD        |        |



## Firmware

The included firmware needs to be flashed to your Electron. It requires 3 libraries, which will automatically be included if you use the Particle CLI compiler or Particle Dev (desktop Atom IDE).

- CellularHelper 0.0.4
- SdFat 0.0.7
- SdCardLogHandlerRK 0.0.4

You can also open [this project in Particle Build WebIDE](https://go.particle.io/shared_apps/5ab4f06005a14f0f79000145).

## Running Tests

If you connect using particle serial monitor, the default options are used. If you want to use a custom APN, keep-alive, or do a cellular tower test, you need to use a terminal program that allows you to send USB serial data, such as:

- PuTTY or CoolTerm (Windows)
- screen (Mac or Linux)
- Particle Dev (Atom IDE) Serial Monitor
- Arduino serial monitor

There's more information on using serial terminals [here](https://github.com/rickkas7/serial_tutorial).

Once you connect to the serial terminal you can press Return or wait a few seconds and you should get a menu:

```
clouddebug: press letter corresponding to the command
a - enter APN for 3rd-party SIM card
k - set keep-alive value
c - show carriers at this location
t - run normal tests (occurs automatically after 10 seconds)
```

If you do nothing, the t option (run normal tests) is run, which behaves like the previous version of cloud debug.

If you press c (show carriers at this location), the program will scan nearby towers and show the carriers, frequencies, and signal strength. This takes several minutes to run, which is why it's not done by default.

If you are using a 3rd-party SIM card, you can set the APN and keep-alive values using the a and k options, respectively.


