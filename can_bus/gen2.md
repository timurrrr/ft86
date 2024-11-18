# CAN bus (2022 model year)

## Pre-word

The purpose of this page is to document as much data channels on the CAN bus
as possible. If your goal is to set up data logging in RaceChrono, there is
a better alternative:
https://github.com/timurrrr/RaceChronoDiyBleDevice/blob/master/can_db/ft86_gen2.md

Special thanks to Ken Houseal for his early legwork on finding and documenting
how to connect to the CAN bus, and finding some of these data mappings!

## Connections

The ODB-II in the driver's footwell no longer exposes the CAN bus data except
for OBD-II requests. If you want to log data with high refresh rates, you need
to use an alternative connection. There are multiple options with different pros
and cons. Below are the recommended ones:

### DCM Connector

**Not available on US market Subaru BRZs.** (Subaru, get your act together?)

GR86s and EU market BRZs have an unused connector hidden behind the glovebox,
near the cabin air filter and the 12V socket:

<img src="../images/gr86_hidden_connector.png"
  alt="Location of the DCM connector" width="500" height="342" />

You can carefully peel off the tape that keeps it out of the way. It's hard but
doable, take your time!

<img src="../images/gr86_dcm_connector.png"
  alt="A look at of the DCM connector" width="500" height="342" />

You can buy a DCM/OBD adapter [from Hachi Electronics](https://hachielectronics.com/products/2022-gr86-brz-asc-can-adapter?variant=49925762810145).

Alternatively, you can make your own harness.
You can use the two middle pins of a
[Toyota radio harness](https://www.amazon.com/gp/product/B0002BEQJ8)
to connect to the CAN bus through that connector.\
Here are the pins that you need to use:

<img src="../images/ft86_socket_wiring.jpg"
  alt="Male connector wiring" width="512" height="384" />

The green wire on the photo is CAN H, the white wire is CAN L.

### ASC port

**This is only recommended for US BRZ owners, as it lacks some data.**

A common place to get access to the CAN bus is the ASC (a.k.a. "fake engine
noise") female connector. It is located inside the dash slightly to the right of
the glovebox, and is accessible by pulling off the panel on the right side of
the dash. The photo below shows the normal location of the connector with an
arrow, and you can see the disconnected connector dangling in the bottom right
of the photo:

<img src="../images/gen2_asc_connector_access.jpg"
  alt="Location of the ASC connector" width="500" height="500" />

You can buy an off-the-shelf harness that taps into the ASC connector and
provides access to the CAN bus in an OBD-II form factor:

* https://hachielectronics.com/products/2022-gr86-brz-asc-can-adapter
* https://ansixauto.com/2022-brz-gt86-can-adapter (has an option to pass through the ASC connections)

Alternatively, you can make your own connector.
You can then buy a male connector by TE Connectivity, part number 1376106-1.
You will also need to buy male pins, part number 1376109-1.

If you're ok with not having the ASC, you need to get at least one connector and
two pins. It might be useful to get some extra in case you fail to crimp the
pins, or something like that.

If you want to keep ASC working, you can also buy a female connector and female
pins, and make a "T" bridge. The part numbers and wiring schematics are out of
scope of this documentation.

Here's how you should wire the male connector:

<img src="../images/gen2_socket_wiring.jpg"
  alt="Wiring for the male ASC connector" width="500" height="500" />

The blue wire on this photo is CAN H, the white wire is CAN L.

The full pinout of the ASC connector is here (with colors matching OEM wire colors):

<img src="../images/gen2_asc_connector_pinout.png"
  alt="ASC connector pinout" width="500" height="115" />

## Details on some CAN IDs

Below are the examples of some high-frequency CAN IDs, along with some of the
data channels that have already been decoded.

Note that the "equation" column in the decoding tables uses the
[RaceChrono equation format](https://racechrono.com/support/equations).

For example, if the data is\
`0x 12 34 56 78 90 AB CD EF`\
where bytes are named\
`-- (A)(B)(C)(D)(E)(F)(G)(H)`,\
then here is how different equations are calculated:

Equation | Value (hex) | Value (dec)
-------- | ----------- | -----------
`A`      | `0x12`      | 18
`B`      | `0x34`      | 52
`F`      | `0xAB`      | 171
`F * 0.1` | N/A        | 17.1
`bytesToIntLe(raw, 0, 2)` | `0x3412` | 13330
`bytesToIntLe(raw, 3, 2)` | `0x9078` | -28552
`bitsToIntLe(raw, 4, 12)` | `0x341`  | 833

You should be able to figure out the format for these equation for other data
logging systems.

### CAN ID 0x40 (64)

Update frequency: 100 times per second.

Example values:\
`0x 5D 0B C9 82 00 00 00 C7` (parked)\
`0x 10 0D FA 06 00 00 00 C3` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
Engine RPM | `bitsToUIntLe(raw, 16, 14)` |
Accelerator position | `E / 2.55` |
Accelerator position | `F / 2.55` | Same value as `E`, until you press and hold both accelerator and brake, then 0.
Accelerator position | `G / 2.55` | Same as `F`.
??? | `H & 0xC0` | `0xC0` when off the accelerator pedal, `0x00` otherwise.

### CAN ID 0x41 (65)

Update frequency: 100 times per second.

Example values:\
`0x C9 43 96 A6 7B 27 65 02` (parked)\
`0x 94 4E 44 A7 78 27 7B 00` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
A/C fan clutch | `H & 0x2` | 2 is engaged, 0 is disengaged

### CAN ID 0x118 (280)

Update frequency: 50 times per second.

Example values:\
`0x 7E 0F 00 07 00 4F 00 00` (parked)\
`0x 8C 02 00 21 00 50 00 00` (moving slowly)

### CAN ID 0x138 (312)

Update frequency: 50 times per second.

Example values:\
`0x 28 0C DD 06 00 00 00 00` (parked)\
`0x 90 0D 13 FA 3F 00 00 00` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
Steering angle | `bytesToIntLe(raw, 2, 2) * -0.1` | Positive value = turning left. You can add a `-` if you prefer it the other way around.
Yaw rate | `bytesToIntLe(raw, 4, 2) * -0.2725` | Calibrated against the gyroscope in RaceBox Mini. Gen1 used 0.286478897 instead.

### CAN ID 0x139 (313)

Update frequency: 50 times per second.

Example values:\
`0x 72 5A 00 E0 08 00 DA 1C` (parked)\
`0x 62 5B FC E0 08 00 CD 1C` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
Speed | `bitsToUIntLe(raw, 16, 13) * 0.015694` | You may want to check the multiplier against an external GPS device, especially if running larger/smaller diameter tires
Brake lights switch | `(E & 0x4)` | `4` is on, `0` is off
Brake pressure | `F * 128` | Coefficient taken from 1st gen cars, but might need to be verified.

When using RaceChrono, you might also consider the following interpretation of the `F` byte:

Channel name | Equation | Notes
------------ | -------- | -----
Brake position (%) | `min(F / 0.7, 100)` | The 0.7 divider seems to be a good value to get 100% at pressure slightly higher than those you're likely to use on the track for cars with no aero. You can use 0.8 or 0.9 if you see 100% too often.


### CAN ID 0x13A (314)

*This channel is not available on the B_CAN which connects to the ASC unit.*

Update frequency: 50 times per second.

Example values:\
`0x 4A 0F 00 00 00 00 00 00` (parked)\
`0x 41 06 00 00 00 00 00 00` (parked)\
`0x 56 CF 0D 7C C1 35 C8 05` (moving at ~6 mph according to the speedometer, turning right)\
`0x FE 75 72 50 0E CC 79 39` (driving on a highway with 65 mph on the speedometer, straight line)

Channel name | Equation | Notes
------------ | -------- | -----
??? | bitsToUIntLe(raw, 0, 12) | Constantly changing even when parked, perhaps some counter?
Wheel speed FL | `bitsToUIntLe(raw, 12, 13) * 0.015694` | Use same multiplier as for speed in 0x139
Wheel speed FR | `bitsToUIntLe(raw, 25, 13) * 0.015694` | Use same multiplier as for speed in 0x139
Wheel speed RL | `bitsToUIntLe(raw, 38, 13) * 0.015694` | Use same multiplier as for speed in 0x139
Wheel speed RR | `bitsToUIntLe(raw, 51, 13) * 0.015694` | Use same multiplier as for speed in 0x139

### CAN ID 0x13B (315)

Update frequency: 50 times per second.

Example values:\
`0x 47 0F 00 00 FF FF FF FF` (parked)\
`0x 37 02 00 00 FF FF FE FD` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
Lateral acceleration | `bytesToIntLe(raw, 6, 1) * 0.2` |
Longitudinal acceleration | `bytesToIntLe(raw, 7, 1) * -0.1` |
Combined acceleration | `sqrt(pow2(bytesToIntLe(raw, 6, 1) * 0.2) + pow2(bytesToIntLe(raw, 7, 1) * 0.1))` |

### CAN ID 0x13C (316)

Update frequency: 50 times per second.

Example values:\
`0x D4 0F 0E 38 02 00 40 00` (parked)\
`0x 12 0E 00 0B 0E 42 6C 00` (moving slowly)

### CAN ID 0x143 (323)

*This channel is not available on the DCM port.*

Update frequency: 50 times per second.

Example values:\
`0x 51 0D 00 00 00 00 00 00` (parked)\
`0x 5C 07 00 10 01 00 00 00` (moving slowly)

Channel name | Equation | Notes
------------ | -------- | -----
Speed | `bitsToUIntLe(raw, 24, 14) * 0.015694` | You may want to check the multiplier against an external GPS device, especially if running larger/smaller diameter tires

### CAN ID 0x146 (326)

Update frequency: 50 times per second.

Example values:\
`0x 7C 1B CE 42 09 01 00 00` (parked)\
`0x 19 11 68 48 11 00 00 00` (moving slowly)

### CAN ID 0x241 (577)

Update frequency: 20 times per second.

Channel name | Equation | Notes
------------ | -------- | -----
Clutch position (%) | `(F & 0x80) / 1.28` | `100` is "clutch pedal depressed", `0` is "clutch pedal released"
Gear | `bitsToUIntLe(raw, 35, 3)` | 0 for N, 1—6 for gears 1–6

### CAN ID 0x2D2 (722)

Update frequency: 33.3 times per second.

Example values:\
`0x 3A 06 40 00 20 00 00 00` (parked)\
`0x 3E 02 48 00 20 00 00 00` (moving slowly)

### CAN ID 0x345 (837)

Update frequency: 10 times per second.

Channel name | Equation | Notes
------------ | -------- | -----
Engine oil temperature | `D - 40` |
Coolant temperature | `E - 40` |

### CAN ID 0x390 (912)

Update frequency: 10 times per second.

Channel name | Equation | Notes
------------ | -------- | -----
Air Temperature | `E / 2 - 40` |

### CAN ID 0x393 (915)

Update frequency: 10 times per second.

Channel name | Equation | Notes
------------ | -------- | -----
Fuel level (%) | `100 - (bitsToUIntLe(raw, 32, 10) / 10.23)` | Unfiltered/noisy. The equation is work in prorgess.

### CAN ID 0x6E2 (1762)

Update frequency: 1 per second.

Example values (from a US car):\
`0x 00 00 04 FE FE FE FE FE` (when pressures aren't displayed shortly after turning ignition on)\
`0x 00 00 04 48 42 46 44 FE` (tire pressures in psi: 36 FL, 33 FR, 35 RL, 34 RR)\
`0x 00 00 24 4E 50 39 4E FE` (tire pressures in psi: 39 FL, 40 FR, 28 RL, 39 RR; RL highlighted yellow, TPMS light on)\
`0x 00 00 24 37 35 48 46 FE` (tire pressures in psi: 27 FL, 26 FR, 36 RL, 35 RR; FL and FR highlighted yellow, TPMS light on)

Channel name | Equation | Notes
------------ | -------- | -----
Tire pressure FL | `lowPass(D >> 1, 126) * 6.8948` | Result is in kPa (as RaceChrono expects)
Tire pressure FR | `lowPass(E >> 1, 126) * 6.8948` | Ditto
Tire pressure RL | `lowPass(F >> 1, 126) * 6.8948` | Ditto
Tire pressure RR | `lowPass(G >> 1, 126) * 6.8948` | Ditto
TPMS light on | `(C >> 5) & 1` |
Low pressure FL | `D & 1` | The FL tire pressure is shown in yellow when this bit is set
Low pressure FR | `E & 1` | Ditto
Low pressure RL | `F & 1` | Ditto
Low pressure RR | `G & 1` | Ditto

Based on my limited testing, the order of tires matches how the pressures are displayed
in the gauge cluster, i.e. correctly account for tire rotation, etc.

These values were read from a US market car where pressures are displayed in psi.
I suspect that cars on other markets (using bar or kPa) will use a different scale
in the data encoding. Please let me know if you find any data examples from non-US
cars and I'll add them here! 

### Typical histogram of CAN IDs

Here's what the distribution of CAN IDs looks like in the CAN bus while idling in a
parking lot for 10 seconds after driving for 10+ minutes:

 CAN ID (hex) | CAN ID (decimal) | Message count | Notes
---- | --- | --- | ---
0x40 | 64 | 1000
0x41 | 65 | 1000
0x118 | 280 | 500
0x138 | 312 | 500
0x139 | 313 | 500
0x13A | 314 | 500 | Not available on the ASC port
0x13B | 315 | 500
0x13C | 316 | 500
0x143 | 323 | 500 | Not available on the DCM port
0x146 | 326 | 500
0x228 | 552 | 167
0x241 | 577 | 200
0x2D2 | 722 | 334 | Not available on the DCM port
0x328 | 808 | 100
0x32B | 811 | 100
0x330 | 816 | 83
0x332 | 818 | 83
0x33A | 826 | 83
0x33B | 827 | 83 | Not available on the DCM port
0x345 | 837 | 100
0x390 | 912 | 100
0x393 | 915 | 100
0x39A | 922 | 50
0x3A7 | 935 | 100 | Not available on the DCM port
0x3AC | 940 | 100
0x3D9 | 985 | 50 | Not available on the DCM port
0x500 | 1280 | 20
0x506 | 1286 | 20 | Not available on the DCM port
0x509 | 1289 | 20 | Not available on the DCM port
0x50B | 1291 | 20 | Not available on the DCM port
0x513 | 1299 | 20 | Not available on the ASC port
0x515 | 1301 | 20 | Not available on the ASC port
0x517 | 1303 | 20 | Not available on the DCM port
0x640 | 1600 | 8 | These few were added after an OTA for the head unit?
0x651 | 1617 | 8 |
0x652 | 1618 | 8 |
0x654 | 1620 | 8 |
0x658 | 1624 | 10 | Not available on the ASC port
0x660 | 1632 | 20
0x663 | 1635 | 10 | These few were added after an OTA for the head unit?
0x68C | 1676 | 8
0x68D | 1677 | 8
0x6B1 | 1713 | 8
0x6BB | 1723 | 8 | Not available on the ASC port
0x6CF | 1743 | 8
0x6DF | 1759 | 8 | Not available on the DCM port
0x6E1 | 1761 | 10
0x6E2 | 1762 | 10
0x6FB | 1787 | 10 | Not available on the ASC port
0x6FC | 1788 | 100 | Not available on the ASC port

---

If you found this page useful, consider donating so I can buy some beer/boba:
 
[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/donate?business=ZKULAWZFJKCES&item_name=Donation+to+support+the+ft86+project+on+GitHub&currency_code=USD)
