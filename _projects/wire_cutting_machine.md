---
layout: page
title: Wire-Cutting Machine
description: 
img: assets/img/wire_cutter_machine_2.JPG
importance: 1
category: work
related_publications: false
mermaid:
  enabled: true
  zoomable: false
pretty_table: true
toc:
  sidebar: left
---

## Motivation
For our [ECE 4180 (Embedded Systems Design)](https://ece.gatech.edu/courses/ece4180) final project, two teammates and I decided to help [HKN](https://hkn.gtorg.gatech.edu/) automate its wire-production process for lab kits. Every semester, as part of a lab-packaging event, members cut and strip over 800 wires, which requires substantial time and effort.

{% include figure.liquid loading="eager" path="assets/img/wire_bundle.jpg" class="rounded mx-auto d-block z-depth-1" max-width="50%" %}
<div class="caption">
    Sample wire-bundle. Contains pairs of 18" red, green, yellow, and black wires
</div>

## System Overview
{% include figure.liquid loading="eager" path="assets/img/wire_cutter_machine_2.JPG" class="img-fluid rounded mx-auto d-block z-depth-1" %}

<div class="caption">
    From left to right: wire cutter with servo motor mechanism, guide tube and servo, stepper motor with extruder, mBED and peripherals on breadboard, uLCD screen
</div>


The system works by unspooling wire into a Bowden extruder using a stepper motor. The wire is fed into a guide tube, which is taped to a servo motor. The servo motor can rotate such that the wire is fed into the notching or cutting part of the wire cutter. Finally, the wire cutter is attached to a DC motor via a wheel contraption. Two limit switches are in place to restrict the wire-cutter's motion.

## Parts List

<table class="table table-sm">
  <thead>
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Quantity</th>
  </tr>
  </thead>
  <tr>
    <td rowspan="3">Cutter</td>
    <td><a href="https://www.sparkfun.com/products/11884">Hitec HS-422 Servo Motor</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td><a href="https://www.sparkfun.com/products/13014">Limit Switches</a></td>
    <td>2</td>
  </tr>
  <tr>
    <td><a href="https://www.sparkfun.com/products/14451">Dual H-Bridge Motor Driver</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td rowspan="4">Feeding</td>
    <td><a href="https://www.amazon.com/Jameco-Reliapro-1124142-Bipolar-Stepper/dp/B07F8YWMQQ">Bipolar Stepper Motor 4.5 VDC, 1500 mA</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td><a href="https://www.pololu.com/product/1182">A4988 Stepper Motor Driver</a></td>
    <td>1</td>
  </tr>
   <tr>
    <td><a href="https://www.amazon.com/Redrex-Upgraded-Aluminum-Extruder-Creality/dp/B07DDGGN921182">Bowden Extruder</a></td>
    <td>1</td>
  </tr>
  
  <tr>
    <td><a href="https://www.sparkfun.com/products/12629">Hall-Effect Wheel Encoder Kit</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td rowspan="1">Guiding</td>
    <td><a href="https://www.sparkfun.com/products/11884">Hitec HS-422 Servo Motor</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td rowspan="3">User Interface</td>
    <td><a href="https://www.sparkfun.com/products/544">microSD Card Reader</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td><a href="https://www.adafruit.com/product/2479">UART Bluefruit LE Module</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td><a href="https://www.sparkfun.com/products/11377">uLCD-144-G2 GFX Display</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td rowspan="1">Microcontroller</td>
    <td><a href="https://os.mbed.com/platforms/mbed-LPC1768/">mBED LPC1768 Microcontroller</a></td>
    <td>1</td>
  </tr>
  <tr>
    <td rowspan="1">Power</td>
    <td><a href="https://www.amazon.com/HiLetgo-Supply-Module-Prototype-Breadboard/dp/B00HJ6AE72">DC Barrel Jack to 3.3V, 5V Board</a></td>
    <td>1</td>
  </tr>
</table>

<p></p>

## RTOS Threads

| Thread | Sleep | Description |
| :------ | :----- | :----------- |
| `updateWireLeft()` | 1s | Update upper half of uLCD: wire left on spool
| `updateBottomScreen()` | 100ms | Update lower half of uLCD: menus
| `saveWireLeft()` | 1 min | Save wire left onto SD card
| `heartbeat()` | 1s | Blink LED4. LED staying on/off indicates error
| `main()` | 100 ms | Run BLE-based state machine

<p></p>

## Bluetooth Control State Machine

Using the [Bluefruit LE Connect App](https://learn.adafruit.com/bluefruit-le-connect/ios-setup), the user is able to control the machine using arrow or number key input. Every key press provides serial data to the Bluefruit module, which triggers an interrupt handler that records the action.

{% include figure.liquid loading="eager" path="assets/img/control_pad.png" class="rounded mx-auto d-block z-depth-1" max-width="50%" %}
<div class="caption">
  Control Pad on Bluefruit LE Connect App
</div>

A small LCD screen provides a menu interface allowing a user to intuitively toggle settings without the parsing hassle of text input. Below is a state diagram of the control flow:

```mermaid
---
title: Menu Flowchart
---
stateDiagram-v2
  direction LR
  [*] --> MENU
  MENU --> CUTTING_ONE:1️⃣
  CUTTING_ONE --> CUTTING_TWO:➡️

  state "SELECT Wire Length" as S1
  state "SELECT L Notch Distance" as S2
  state "SELECT R Notch Distance" as S3
  state "SELECT Number of Wires" as S4
  state "INCR setting" as SU
  state "DECR setting" as SD

  CUTTING_ONE --> S1:1️⃣
  S1 --> CUTTING_ONE
   CUTTING_ONE --> S2:2️⃣
  S2 --> CUTTING_ONE
   CUTTING_ONE --> S3:3️⃣
  S3 --> CUTTING_ONE
   CUTTING_ONE --> S4:4️⃣
  S4 --> CUTTING_ONE
    CUTTING_ONE --> SU: ⬆️
  SU --> CUTTING_ONE
    CUTTING_ONE --> SD
  SD --> CUTTING_ONE: ⬇️

  state "Cut/Notch Wires" as CUTTING_TWO
  CUTTING_TWO --> MENU
  MENU --> SETTINGS_ONE:2️⃣
  SETTINGS_ONE --> MENU:⬅️
  SETTINGS_ONE --> SETTINGS_RESET:1️⃣
  SETTINGS_RESET --> SETTINGS_ONE
  SETTINGS_ONE --> SETTINGS_FEED:2️⃣
  SETTINGS_FEED --> SETTINGS_ONE:⬅️


  state "Manually feed wire FWD" as SFA
  SETTINGS_FEED --> SFA: ⬆️
  SFA --> SETTINGS_FEED
   state "Manually feed wire BWD" as SFB
  SETTINGS_FEED --> SFB: ⬇️
  SFB --> SETTINGS_FEED

  SETTINGS_ONE --> SETTINGS_GUIDE:3️⃣
  SETTINGS_GUIDE --> SETTINGS_ONE:⬅️
    
  state "Manually turn guide CW" as SGA
  SETTINGS_GUIDE --> SGA: ⬆️
  SGA --> SETTINGS_GUIDE
   state "Manually turn guide CCW" as SGB
  SETTINGS_GUIDE --> SGB: ⬇️
  SGB --> SETTINGS_GUIDE


  SETTINGS_ONE --> SETTINGS_CUTTER:4️⃣
  SETTINGS_CUTTER --> SETTINGS_ONE:⬅️
  
   state "Manually open cutter" as SCA
  SETTINGS_CUTTER --> SCA: ⬆️
  SCA -->  SETTINGS_CUTTER
   state "Manually close cutter" as SCB
  SETTINGS_CUTTER --> SCB: ⬇️
  SCB -->  SETTINGS_CUTTER
```
## Code
<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="tcontis/Wire_Cutter_Machine" %}
</div>

## Demo
<div  class="rounded mx-auto d-block" style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 75%; height: auto;">
  <iframe 
    src="https://www.youtube.com/embed/ptiaEEuiNBE"
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;">
  </iframe>
</div>

<p></p>

## Results & Potential Improvements
Despite the smooth interface and sufficient precision of 0.2", the machine's servo guide and wire-cutting mechanisms suffer from issues.

Firstly, the 3D-printed wheel attached to the DC motor shaft often comes loose in motion, despite using a wedge. Introducing a screw into the shaft could fix this.

Secondly, the servo lacks precision and consistency when guiding the wire into the cutting or notching portions of the wire cutter. Mechanically, the tube requires better stabilization. On the software-side, the servo should use closed-loop control to ensure precision.

