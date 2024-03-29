# The ShockCell Project

## Overview

This project is about creating an internet-controlled electronic key safe with additional control functions. The key safe uses Wi-Fi to connect to the internet and provides a Telegram messenger chat bot for interaction with remote users.

A major additional function of the key safe is to trigger shocks using a shock device for remote controlled punishments.

## Use Case

Chastity play between a remote holder and a chastity device wearer requires secure management of the keys allowing to unlock the chastity device. This key safe is meant to serve the purpose of allowing the holder to decide when keys are returned to the wearer.

I use a chastity assembly consisting of a special chastity cage with an integrated shock device and an electronically controlled, Wi-Fi enabled key safe for my chastity keys with an integrated remote control for the shock device. With this project, I want to give others the opportunity to do the same or exchange ideas and improve the concept.

## Description

The key safe consists of a flat box with a hinged cover that automatically locks once pressed shut. It incorporates a microcontroller board hosting the entire software of this project. The microcontroller board connects to several sensors and actors:
* Actor activating the keypad button of the remote control for the shock device,
* servo motor for pulling open the locking bracket of the cover ratchet mechanism,
* sensor of the state of the cover (open/closed) and
* display for information about the current state of the key safe.
The key safe encloses a compartment for storing the keys as well as all other components.

It is connected to the LAN as a Wi-Fi based client and powered over USB.

## Shocker description (light version)

Instead of the full safe with integrated shocker control, a light version with the shocker control only can be realized. In this version, the internet connectivity and app control is identical. The advantage is significantly reduced complexity building this.

The schematic and board design is provided below labeled as "light" version.

## Operation

To begin a session, you put all the chastity device keys into the key safe and close it. The ratchet mechanism keeps the lock box now closed until a valid unlock command is sent to the Telegram bot.

***
# How to prepare the software for use with the Arduino IDE

- Rename the file Defs_template.h to Defs.h
- This is necessary to customize the creadentials for Wi-Fi/WLAN access und connecting the Telegram bot.

# How to do the Telegram setup

## Telegram bot

The ShockCell key safe runs a bot for the Telegram Messenger App. The bot is the only interface to the key safe. It can be added to a group chat to control the key safe. While the group chat can be used for general communication between the holder, wearer and other users. the following commands are available to control the key safe. Any command must be send as a separate message and should not contain any other text. The prefix for each command is a slash "/".

## Create the bot

https://telegrambots.github.io/book/1/quickstart.html

### Result
This step provides as a result the bot authentication token. It has the following pattern:

1234567:4TT8bAc8GHUspu3ERYn-KGcvsvGB9u_n4ddy

This token is needed in the source code in file Defs.h

### Roles

The ShockCell key safe knows four different roles in the group chat.

1. Wearer - the one person being controlled and wearing a chastity device with a lock and shock device.
2. Holder - the one person controlling (holding) the key safe storing the keys to that lock. She/he also controls the shocks for the wearer.
3. Guest - additional people in the group chat without special rights
4. Teaser - additional people who support the holder with the rights to send shocks to the wearer (this right can be controlled by the holder).

Every member in the group chat can have only one role. However, the following role transitions are possible.

| New role: | -> Holder                            | -> Teaser                                  | -> Guest                                   | -> Wearer               |
|-----------|--------------------------------------|--------------------------------------------|--------------------------------------------|-------------------------|
| Holder    | -                                    | yes, setting the wearer free automatically | yes, setting the wearer free automatically | no                      |
| Teaser    | yes, if there is currently no holder | -                                          | yes                                        | no                      |
| Guest     | yes, if there is currently no holder | yes                                        | -                                          | no                      |
| Wearer    | no                                   | no                                         | no                                         | -                       |

#### Rules for role changes

- The wearer cannot change roles.

- A guest may become holder if there is currently no holder. The holder can become guest again and give up his privileges.

- A guest may become teaser. A teaser can become guest again and give up his privileges.

### Start communication/overview

#### /start

The /start command lists all commands available for the role 

#define BOT_COMMANDS_GENERAL "/start - Start communication\n/state - Report the cover state\n/roles - List roles\n/users - List users in chat\n"
#define BOT_COMMANDS_SHOCKS "/shock1 - Shock for 1 seconds\n/shock3 - Shock for 3 seconds\n/shock5 - Shock for 5 seconds\n/shock10 - Shock for 10 seconds\n/shock30 - Shock for 30 seconds\n"
#define BOT_COMMANDS_RANDOM "/random_5 - Switch on random shock mode with 5 shocks per hour (other intervals work with corresponding numbers)\n/random_off - Switch off random shock mode\n"
#define BOT_COMMANDS_TEASING "/teasing_on - Enable teasing\n/teasing_off - Disable teasing\n"
#define BOT_COMMANDS_UNLOCK "/unlock - Unlock key safe\n"
#define BOT_COMMANDS_ROLES "/holder - Adopt holder role\n/teaser - Adopt teaser role\n/guest - Adopt guest role\n"
#define BOT_COMMANDS_WAITING "/waiting - Make wearer waiting to be captured by the holder\n/free - Make wearer free again (stops waiting for capture)\n"
#define BOT_COMMANDS_CAPTURE "/capture - Capture wearer as a sub\n/release - Release wearer as a sub"
#define BOT_COMMANDS_EMERGENCY "/thisisanemergency - Release the wearer in case of an emergency\n"

***
# How to setup the components of the light version of this project

![Schematic](https://github.com/cb-lock/ShockCell/blob/1a3034a8b6907565f01edb1c4ef63b5c6e9200a3/ShockCell%20light_Schaltplan.png?raw=true)

![Board design](https://github.com/cb-lock/ShockCell/blob/main/ShockCell%20light_Steckplatine.png?raw=true)

## Hardware components

The key safe consists of mechanic hardware and electronics. The mechanics are probably something individual, but I would like to share the electronics part listing and schematics.

### Parts Listing

- ESP32 board. Probably any ESP32 board will do. I use the DOIT ESP32 DEVKIT V1. It is available from different suppliers. I got mine from Aliexpress, https://de.aliexpress.com/item/32992390199.html
- 0,96 zoll IIC display - I2C interface seems easier to me, but the U8g2 library supports a wide range of display types. Mine is this: https://de.aliexpress.com/item/32899149458.html
- Transistor BC547 - needed to activate the button of the remote control for the shock device (shock collar)
- 10 kOhm resistor for the transistor base
- 
- Dog shock collar - Any truly waterproof device will do with a good range and a remote control with buttons. I use https://de.aliexpress.com/item/32915272728.html
- A switch sensor - needed to detect if the key safe is open or closed
- USB cable for the power supply (micro USB)

## Software components

I use the Arduino ISE to build and uploaded the project to the device. The software of this project relies on the following libraries:

- ArduinoJson - from Arduino library
- UniversalTelegramBot by Brian Lough  - with my modifications, see https://github.com/cb-lock/Universal-Arduino-Telegram-Bot

# Reference
- ESP32 tutorial - https://randomnerdtutorials.com/getting-started-with-esp32/
- Connecting ESP32 to display - http://www.iotsharing.com/2017/05/how-to-use-arduino-esp32-to-display-oled.html?m=1
- U8g2 library - https://github.com/olikraus/u8g2/wiki/setup_tutorial

