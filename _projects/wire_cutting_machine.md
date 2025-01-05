---
layout: page
title: Wire-Cutting Machine
description: 
img: assets/img/wire_cutter_machine_2.JPG
importance: 1
category: work
related_publications: false
---

## Motivation
For our [ECE 4180 (Embedded Systems Design)](https://ece.gatech.edu/courses/ece4180) final project, two teammates and I decided to help [HKN](https://hkn.gtorg.gatech.edu/) automate its wire-production process for lab kits. Every semester, as part of a lab-packaging event, members cut and strip over 800 wires, which represents a lot of time and effort.

## System Design
{% include figure.liquid loading="eager" path="assets/img/wire_cutter_machine_2.JPG" class="img-fluid rounded z-depth-1" %}

```mermaid
---
title: Menu Flowchart
---
stateDiagram-v2
  direction LR
  [*] --> MENU
  MENU --> CUTTING_ONE:1️⃣
  CUTTING_ONE --> CUTTING_TWO:➡️
  CUTTING_TWO --> MENU:➡️
  MENU --> SETTINGS_ONE:2️⃣
  SETTINGS_ONE --> MENU:⬅️
  SETTINGS_ONE --> SETTINGS_RESET:1️⃣
  SETTINGS_RESET --> SETTINGS_ONE
  SETTINGS_ONE --> SETTINGS_FEED:2️⃣
  SETTINGS_FEED --> SETTINGS_ONE:⬅️
  SETTINGS_ONE --> SETTINGS_GUIDE:3️⃣
  SETTINGS_GUIDE --> SETTINGS_ONE:⬅️
  SETTINGS_ONE --> SETTINGS_CUTTER:4️⃣
  SETTINGS_CUTTER --> SETTINGS_ONE:⬅️
```
## Demo
{% include video.liquid path="https://www.youtube.com/embed/ptiaEEuiNBE" class="img-fluid rounded z-depth-1" %}
