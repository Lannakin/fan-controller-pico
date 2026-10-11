<!-- ./README.md -->
<!-- markdownlint-disable MD013 MD032 MD007 -->
# **Overview** - Fan Controller TXU0204 Pico
<!-- quick summary for github -->
⚠️⚠️⚠️ WARNING ⚠️⚠️⚠️

i am just making this for a hobby.  i have no idea what i am doing.  as you can
see from my profile picture, i am obviously a cat, which means my electrical and
maths skills in general are questionable *at best*.

⚠️⚠️⚠️ WARNING ⚠️⚠️⚠️

i hope github completely mangles my indentation.

using the gratuitously over-qualified raspberry pi pico series, this expansion
board is meant to control up to 4 fan connections **that had better not exceed
1.5a.** they should be 12vdc as well.

pwm output from pico is shifted from 3.3vdc to 5vdc because if i used a 12vdc
noctua fan, that's what noctua expects.  intel recommended supporting 3.3vdc
logic, but that's just a recommendation in what seems to be a de-facto standard,
not an industry requirement.

the current immediate usage scenario is controlling an air filtration box using
a total of 6 [arctic p12 pro pst ln fans](https://www.arctic.de/media/03/4b/1a/1771239441/Spec_Sheet_P12_Pro_PST_LN_EN.pdf).  these fans are daisy-chainable, with
the power and pwm chained to every fan in the series, but (obviously) only the
first fan's tachometer signal connected.

however, i would like this design to support more than 2 channels of fan pwm and
fan tachometer, so that it can be used on other projects.

---

## **CREDITS FOR FONTS**

[WebPlus\_ToshibaSat\_8x14 and WebPlus\_ToshibaSat\_9x16 by VileR](https://**int10h**.org/oldschool-pc-fonts/)

## **Schematic Notes**
<!-- Key circuit blocks, design rationale -->
### expected fan needs gleaned from various documentation

power 12vdc:
12vdc (less than 12.6vdc) power to fan
* current draw per fan probably no higher than 0.7a

power 5vdc:
* rp2040 and rp2350 draw < 0.2A (the datasheets' highest charts were like 37mW
  for the rp2350, and [94.1mA for the pico 2 board from vbus](https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008307-DS-2-pico-datasheet.pdf#page=13).  these datasheets
  kinda suck)
    * for [cyw43439](https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/7160/CYW43439KUBGT.pdf), probably around 0.5A
* txu0204 power draw

pwm signal:
* 5vdc pullup, max 5.25vdc
* should be open-collector
* max current 5mA, but device should be capable of 8mA minimum
* 21 - 28kHz is the range for arctic's accepted quality control; 25kHz preferred

tach signal:
* usually pulled up using 12vdc
* noctua says max of 13v, but fan manufacturers probably only expect 5vdc
  can be pulled up to 5vdc or 3.3vdc
* probably open-collector
* max current 5mA, but device should be capable of 8mA minimum
* 2 pulses per revolution

## **Layout Considerations**
<!-- Layer stackup, impedance, keep-outs -->
might be 2 layer
should really be 4 layer sig / gnd / pwr / sig
* oshpark

general routing
* every trace will "try" to return to ground using shortest possible path under?
  the trace
* try to avoid splitting ground planes, and use vias to help keep ground planes
  contiguous
* every time trace uses a via to switch layers, add via to ground as close as
  possible to give the voltage a return shorter path

signal routing:
* if having to cross another signal on another layer, try to do so
  perpendicularly
* try to keep the traces as spread out as possible
* larger traces have better noise rejection

## **Component Notes**
<!-- Click on designators like R1, C5, U3 to highlight on PCB -->
other signal translation component possibilities:
SN74LVC2G17
TXU0204 - integrated pull-down resistors
TXU0202 - integrated pull-down resistors

### TXU0204

"4-Bit Fixed Direction Voltage-Level Translator with Schmitt-Trigger Inputs and
3-State Outputs"

* vcc determines output voltage level
* using 2 to convert pico-output 3v3 pwm signals to 5v fan-input
* using 2 to convert fan-output 5v tachometer signals to 3v3 pico-input
* integrated 5Mohm pull-down resistors

## **Power Distribution**
<!-- Power rails, decoupling strategy -->
syntax of subsections:

```
### (heading size 3) Xvdc rail

device / component on the rail

* note about that device / component
* another note on the device / component
    * subnote on another note
    * another subnote on another note
* yet another note on that device / component
```

### general rail stuff

regarding rail traces / zones in general

* keep rail traces and zones off to the side as much as possible
* attempt to give them clear ground returns with gratuitous vias
* the zones for the positive inputs / outputs are mostly for heat sinking; the
  small size of the traces are plenty for these low voltages and amperages

### 12vdc rail

chanzon [2abn036f](https://www.amazon.com/dp/B07HNV6SBJ) 120vac 1a to 12vdc 3a wall wart:

* claims to be UL listed and shit but i highly doubt it is
* powers pico and this expansion board
* powers 4 x [P12 Pro PST](https://support.arctic.de/p12-pro-pst/docs) @ up to 0.3a each
* powers 1 x [OKI-78SR-5/1.5-W36-C](https://pim.murata.com/asset/pim4/nonIsolatedDCDCconverter/OKI-78SR_PDF_NONISOLATEDDCDCCONVERTER) @ 0.69a
* desired circuit protections: transient, reverse polarity, and overcurrent
  protection; ~~maybe also overvoltage and undervoltage~~ space constraints
    * tvs diode: SMAJ16CA
    * ~~efuse choice 1: [TPS259482A](https://www.ti.com/lit/gpn/TPS25948)~~ space constraints
        * protects against overvoltage, overcurrent, shorts, reverse current,
          undervoltage
    * ~~efuse choice 2: [TPS259472A](https://www.ti.com/lit/ds/symlink/tps25947.pdf)~~ space constraints
        * protects against overvoltage, overcurrent, shorts, reverse current,
          reverse polarity, undervoltage
    * reverse polarity protection
        * p-mosfet
    * input tvs diodes on main power and +12vdc fan pins
        * bidirectional to protect against surges from GND
    * current limit
    * input limit <=2A 12vdc to 5vdc psu
    * idk for fans
* may need filtering but i don't own a real oscilloscope soooooooooo

OKI-78SR-5/1.5-W36-C (12vdc input):

* based on [MP2467](https://www.monolithicpower.com/en/documentview/productdocument/index/version/2/document_type/Datasheet/lang/en/sku/MP2467/document_id/263/)
* recommends 2a fuse
* **inrush transient**: 0.16 A2-Sec.
* might need an electrolytic capacitor for up to 3ft of "20awg" wire from the
  wall wart to prevent instability or whatever
* no mention of needing input filtering;  that's very bold of you, murata
* the only filtering mentioned appears to be what is physically on the module's
  pcb: 2x 100uF (ceramic capacitors)
* desired circuit protections: counter-emf schottky diode
    * schottky diode SGL1-20 for protection against counter-emf when power is
      shut off if fuse blows / power is lost
    * output of 12vdc to 5vdc psu has built-in limiter
* with supposed ripple of 30mV peak-to-peak, doubt additional filtering is
  neeeded

### 5vdc rail

OKI-78SR-5/1.5-W36-C (5vdc output):

* output of 5vdc at up to 1.5a
* mostly selected because it's in my drawer from years ago and has like <30mV
  ripple, which appears to be completely obscene even at like $6
* the okami series that this dcdc module is from has a cool wolf logo
* no mention of adding additional filtering; output filtering on the module pcb
  (assuming that's what "Cvbus" is) is 1x 1000uF (ceramic capacitor)

pico supplying via vsys

* raspberry pi pico datasheets [rec using p-mosfet DMG2305UX or a schottky diode](https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008307-DS-2-pico-datasheet.pdf#page=21)
    * ORing circuit for 5vdc vbus < - > vsys
        * diode - feel like Vf drop too high, might hinder fans' pwm input
        * p-mosfet - i ain't wanna do another one of these
        * ORing dual ideal diode
        * lm66200 is like \$0.60 each max, cool, simple, and small, vs p-fet at
          like \$0.45 each, 40 mΩ on-resistance

### 3.3vdc rail

pico 5vdc to 3.3vdc converter (same for pico, w, 2, 2w): [RT6150](https://www.richtek.com/assets/product_file/RT6150A=RT6150B/DS6150AB-06.pdf)

* note that the datasheet won't fully cover the converter's output capabilities
  b/c the converter is the sum of the conversion circuit
* [super noisy in power save mode](https://pico-adc.markomo.me/PSU-Noise/) (115mV pk-pk vs 31mV); power efficiency is
  much worse, though
* maybe should use linear regulator
* if using analog, rec use external shunt ref like [lm4040](https://www.ti.com/lit/ds/symlink/lm4040.pdf) (in case using
  potentiometer for fan speed setting)
    * note: someone said for a human-operated potentiometer, the precision
      probably doesn't matter
* recommended: stay under 300mA

## **Signal Integrity**
<!-- Critical nets, routing constraints -->
this should probably be a 4-layer board, i don't think i can avoid splitting the
ground plane.

## **References**
<!-- Datasheets, application notes, calculations -->
### pcb production

oshpark:
  [oshpark - 2 layer prototype service](https://docs.oshpark.com/services/two-layer/)
  [oshpark - after dark (also 2 layer) service](https://docs.oshpark.com/services/afterdark/)
  [oshpark - 4 layer prototype service](https://docs.oshpark.com/services/four-layer/)

digikey:
  [pcb builder - faq](https://www.digikey.com/en/resources/design-tools/pcb-faqs)
  [pcb builder - technical specs](https://www.digikey.com/en/resources/design-tools/pcb-builder-technical-specs)

### efuses

[texas instruments "basics of efuses"](https://www.ti.com/lit/an/slva862a/slva862a.pdf)

### general 12vdc pwm fan control (pc fans)

[intel corporation "4-wire pulse width modulation (pwm) controlled fans"](https://glkinst.com/cables/cable_pics/4_Wire_PWM_Spec.pdf)
  reference to pulling up to 12v power to fans on tachometer pin

[edn article "4 wire pc fan"](https://www.edn.com/4-wire-pc-fan/)
[yoinked motherboard schematic picture from edn article](https://www.edn.com/wp-content/uploads/2019/12/2-Fan-Connector-Scheme.png)
  pulls up to 12v power to fans on tachometer pin

[texas instruments "fan controller overview"](https://www.ti.com/lit/po/sprt839/sprt839.pdf)
  "The tachometer output from the fan is
  typically an open-collector (or open-drain) output that requires a pull-up
  resistor. It's customary to connect this pull-up to the same voltage as the fan
  power."
  "you can still pull up to a lower voltage though" (slight paraphrasing)

### arctic p12 pro pst (ln)

[pwm curve for p12 pro pst series](https://support.arctic.de/products/p12-pro-pst/techdocs/P12%20Pro%20Series_PWMcurve.pdf)
[p12 pro pst docs page](https://support.arctic.de/p12-pro-pst/docs)
[p12 pro pst pwm guidelines](https://support.arctic.de/p12-pro-pst#toc-103)
[p12 pro pst ln docs page](https://support.arctic.de/p12-pro-pst-ln/docs)
[p12 pst pro ln pwm guidelines](https://support.arctic.de/p12-pro-pst-ln#toc-107)

### noctua fan jumpscare

[noctua "noctua pwm specifications white paper"](https://cdn.noctua.at/preview/media/Noctua_PWM_specifications_white_paper.pdf) Vtachometer for 12V fans: 5V
  recommended (13V max.)" at <5mA for 12v fans

### raspberry pi pico boards

[pico power supply](https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008307-DS-2-pico-datasheet.pdf#page=21)
  note: this applies to pico, pico w, pico 2, pico 2w

---

### layout filched from [KiNotes](https://pcbtools.xyz/tools/kinotes)
