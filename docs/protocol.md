# MQTT packets sent by the Atari

FujiNet's N: device has no MQTT support (despite what Gemini claimed), but
it speaks raw TCP - so the Atari sends hand-built MQTT 3.1.1 packets byte
by byte with `PUT #1,<byte>`. All values below are decimal, exactly as in
`atari/mqtt_toggle.bas`. Every press of RETURN opens a new TCP connection,
sends CONNECT + one PUBLISH, and closes it again.

## CONNECT (lines 140-149, 31 bytes)

| Pos | Value | Meaning |
|---|---|---|
| 1 | 16 | Packet type CONNECT (0x10) |
| 2 | 29 | Remaining length: 29 more bytes follow |
| 3-4 | 0, 4 | Length of the protocol name |
| 5-8 | 77, 81, 84, 84 | "MQTT" |
| 9 | 4 | Protocol level 4 = MQTT 3.1.1 |
| 10 | 194 | Connect flags 0xC2: username + password follow, clean session |
| 11-12 | 0, 60 | Keep-alive 60 seconds |
| 13-14 | 0, 0 | Client ID length 0 - the broker assigns an ID itself |
| 15-16 | 0, 8 | Username length 8 |
| 17-24 | 116, 101, 115, 116, 117, 115, 101, 114 | "testuser" |
| 25-26 | 0, 5 | Password length 5 |
| 27-31 | 97, 116, 97, 114, 105 | "atari" |

To use a different user, re-encode the username/password bytes **and**
adjust both length fields and the remaining length in position 2
(29 = 10 + 2 + 2 + username length + 2 + password length).

## PUBLISH "OFF" (lines 190-193, 14 bytes)

| Pos | Value | Meaning |
|---|---|---|
| 1 | 49 | Packet type PUBLISH (0x30) + RETAIN flag (0x01), QoS 0 |
| 2 | 12 | Remaining length: 12 more bytes follow |
| 3-4 | 0, 7 | Topic length 7 |
| 5-11 | 65, 116, 97, 114, 105, 88, 76 | "AtariXL" |
| 12-14 | 79, 70, 70 | Payload "OFF" |

## PUBLISH "ON" (lines 220-223, 13 bytes)

| Pos | Value | Meaning |
|---|---|---|
| 1 | 49 | PUBLISH + RETAIN, QoS 0 |
| 2 | 11 | Remaining length: 11 more bytes follow |
| 3-4 | 0, 7 | Topic length 7 |
| 5-11 | 65, 116, 97, 114, 105, 88, 76 | "AtariXL" |
| 12-13 | 79, 78 | Payload "ON" |

(The original write-up's ON table says "12 bytes follow" - a typo there;
the listing correctly sends 11.)

Because RETAIN is set, the broker keeps the last value, so Home Assistant
(or any later subscriber) sees the current state even if it connects after
the Atari sent it.

## What's not implemented

- The CONNACK (32, 2, 0, 0) the broker sends back is never read.
- No DISCONNECT packet (224, 0) is sent before `CLOSE #1` - the broker
  just logs the connection as closed, which works fine.
- QoS 0 only, so there's no PUBACK to wait for either.
