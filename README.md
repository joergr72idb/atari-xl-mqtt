# Atari XL MQTT

An Atari XL/XE with FujiNet publishes an MQTT topic to Home Assistant,
straight from Atari BASIC. Pressing RETURN toggles the topic `AtariXL`
between `ON` and `OFF`; Home Assistant sees it as a binary sensor that
can drive automations (e.g. switch a lamp on or off).

FujiNet's [N: device](https://fujinetwifi.github.io/fujinet-docs/features/network-device/)
supports HTTP, HTTPS, FTP, SSH, TCP and more, but **not** MQTT. So the
program opens a plain TCP connection to the broker and sends the MQTT
packets itself, byte by byte - see [`docs/protocol.md`](docs/protocol.md)
for what every byte means.

## Files

```
atari/mqtt_toggle.bas          Atari BASIC program (the client)
homeassistant/configuration.yaml   binary_sensor snippet for Home Assistant
docs/protocol.md               byte-by-byte breakdown of the MQTT packets
docs/MQTT_writeup_de.pdf       original write-up (German), source of this project
docs/MQTT_writeup_en.pdf       English version of the write-up, with corrections
docs/writeup_en/               its HTML source (print to PDF with headless Chrome)
```

## Setup

1. **Broker:** a Mosquitto broker, e.g. the Mosquitto app (formerly
   add-on) in Home Assistant OS. [MQTT Explorer](https://mqtt-explorer.com/)
   is handy for watching the topic.
2. **User:** the program logs in as `testuser` with password `atari`.
   Create that user in Home Assistant or in the Mosquitto app, and check
   with MQTT Explorer that it can log in. To use a different user, see
   `docs/protocol.md` (lengths must be adjusted, not just the letters).
3. **Atari:** set the broker's IP address in line 10
   (`IP$="N:TCP://<ip>:1883/"`), then load the program. Easiest way:
   paste it into Atari BASIC in Altirra with FujiNet, save it, and copy
   the disk image to the real FujiNet's SD card.
4. **Home Assistant:** add the snippet from
   `homeassistant/configuration.yaml` to your `mqtt:` block and restart
   Home Assistant Core. The sensor "Atari XL Button" then appears.

Press RETURN on the Atari - it prints `SENDE STATE: ON` / `OFF` and the
sensor follows.

## Status

- Tested on a real, unmodified Atari XL + FujiNet against Home
  Assistant. Developed on a notebook with a HAOS VM,
  [Altirra](https://virtualdub.org/altirra.html) and the
  [FujiNet emulator bridge](https://github.com/FujiNetWIFI/fujinet-emulator-bridge).
- 2026-09-27: the exact byte sequences from the listing were replayed
  against a local Mosquitto 2 broker - CONNECT accepted (broker-assigned
  client ID), both retained PUBLISHes stored and delivered to a later
  subscriber. (That test broker ran without a password file, so the
  credential check itself wasn't part of it.)

## Ideas

- Read the broker's CONNACK instead of ignoring it.
- Use the joystick's fire button instead of RETURN.
- A back channel (SUBSCRIBE) to show Home Assistant states on the Atari -
  an 8-bit control panel for the retro computer room.
- Port to other machines that can send raw TCP bytes: Schneider/Amstrad
  CPC with M4, C64 with Meatloaf.

## Credits

Written by joergr ([Classic Computing forum](https://forum.classic-computing.de/forum/),
VzEkC e.V.), with help from AI (Gemini) - the write-up itself was mostly
done by hand. This repository was assembled from that write-up with
Claude.
