# CLAUDE.md - Atari XL MQTT

Small project: Atari BASIC + FujiNet N: device (raw TCP) sending
hand-built MQTT 3.1.1 packets to a Mosquitto broker for Home Assistant.
See README.md for setup and docs/protocol.md for the packet bytes.

- **Source of truth:** `docs/MQTT_writeup_de.pdf` (the user's original
  German write-up). `atari/mqtt_toggle.bas` is a verbatim transcription
  of its listing - German REMs/PRINTs and the `<IP-Adresse einsetzen>`
  placeholder kept as-is. There is no line 260 in the original (250 REM
  is followed directly by 270); that's not a transcription gap.
- **When changing any PUT bytes**, keep the MQTT length fields
  consistent (remaining length in byte 2, string length prefixes) - see
  docs/protocol.md. A quick check: decode the PUT values from the
  listing and compare byte 2 against the actual packet length.
- **Testing without hardware:** `mosquitto -p 18830 -v` (no config
  needed) and replay the listing's bytes over a Python socket, then
  SUBSCRIBE to `AtariXL` to see the retained value. Done on 2026-09-27,
  both ON and OFF worked.
- **Transfer to the Atari:** same as in
  [`retro-tron-battle/CLAUDE.md`](../retro-tron-battle/CLAUDE.md)
  ("Transferring clients to the target systems" -> Atari): paste into
  Altirra with FujiNet, save, copy the image to the real SD card.
