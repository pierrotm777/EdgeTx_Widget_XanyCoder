
# XANYCTL Widget for EdgeTX (Radiomaster TX16S)

## Overview

**XANYCTL** is a touchscreen widget for **EdgeTX** designed to control a **Multiplex X‑Any encoder** using Lua.  
It provides a graphical interface (buttons + slider) that allows the pilot to control **up to 16 logical switches** and an optional **PROP analog parameter**.

The widget is intended to work with a companion **Mix script** that converts the widget state into a valid **X‑Any pulse stream** sent on a radio channel.

Typical use case:

TX16S → EdgeTX widget → Lua Mix script → RC channel → receiver → X‑Any decoder.

The project is composed of two main parts:

* **Widget UI (WIDGETS/XANYCTL)** – graphical interface and state management
* **Mix script (SCRIPTS/MIXES/xanytx.lua)** – X‑Any protocol generation


---

# Architecture

```
SDCARD/
 ├─ WIDGETS/
 │   └─ XANYCTL/
 │        ├─ main.lua
 │        ├─ buttons.lua
 │        └─ README.md
 │
 └─ SCRIPTS/
     └─ MIXES/
          └─ xanytx.lua
```


## main.lua

This is the **widget entry point**.

Responsibilities:

* Define **widget options**
* Provide the **API used by the UI**
* Store and retrieve values from **EdgeTX Global Variables (GVARS)**
* Initialize the GUI and load configuration

The widget does **not generate X‑Any frames directly**.  
It only stores user inputs in GVars which are later read by the mix script.



## buttons.lua

Contains the **graphical interface** using **libGUI**.

Features:

* Toggle buttons
* Momentary buttons
* Vertical PROP slider
* Rounded UI elements
* Optional shadows
* Custom ON/OFF colors
* Touch interaction

The UI is intentionally separated from `main.lua` to keep the architecture modular.



## xanytx.lua

Lua **Mix Script** responsible for generating the X‑Any signal.

Responsibilities:

* Read widget state from **GVars**
* Build the **X‑Any payload**
* Compute checksum
* Apply **R compression**
* Handle **Repeat**
* Convert nibbles to **EdgeTX pulse widths**
* Output signal on the assigned RC channel



---

# Data Storage (GVars)

The widget uses **EdgeTX Global Variables** to exchange data with the mix script.

| GVar | Purpose |
|-----|--------|
| GV1 | Switch mask (low bits) |
| GV2 | Switch mask (high bits) |
| GV3 | Repeat value |
| GV4 | Mode |
| GV5 | Channel memory |
| GV6 | PROP value (0‑255) |


### Switch Mask

16 switches are packed into a **16‑bit mask**.

```
mask = GV1 + GV2 * 2048
```

Each bit represents a switch state.



---

# Supported Modes

| Mode | Description |
|-----|-------------|
| 0 | SW8 |
| 1 | SW8 + PROP |
| 2 | SW16 |
| 3 | SW16 + PROP |



---

# X‑Any Protocol

The X‑Any protocol sends **nibbles encoded as pulse widths**.

### Symbols

| Symbol | Meaning |
|------|---------|
| 0‑15 | nibble values |
| R | repeat previous nibble |
| I | idle |

### Typical pulse widths

| Symbol | Pulse |
|------|-------|
| 0 | ~1024 µs |
| 5 | ~1300 µs |
| R | ~1912 µs |
| I | ~1976 µs |



---

# Frame Structure

```
I
DATA_HI
DATA_LO
CHECKSUM_HI
CHECKSUM_LO
I
```

Checksum:

```
checksum = data XOR 0x55
```



---

# Compression

Repeated nibbles are compressed using the **R symbol**.

Example:

```
5 5 0 0
→ 5 R 0 R
```



---

# Repeat Mechanism

If Repeat = N, the compressed sequence is repeated N times before idle.

Pipeline:

```
RAW nibbles
↓
R compression
↓
Repeat
↓
Idle
```



---

# User Interface

The UI uses **libGUI** components:

* Rounded buttons
* Toggle and momentary actions
* Vertical slider
* Customizable colors
* Optional shadows

The slider controls the **PROP value (0‑255)** and displays the percentage.



---

# Installation

## 1. Copy Files

Copy the folders to your **EdgeTX SD card**.

```
SDCARD/WIDGETS/XANYCTL/
SDCARD/SCRIPTS/MIXES/xanytx.lua
```


## 2. Add Widget

1. Open your model
2. Go to **Display → Widgets**
3. Select an empty slot
4. Choose **XANYCTL**


## 3. Configure Options

Available widget options:

* MODE
* CH
* Repeat
* OffCol
* OnCol
* Shadow


## 4. Add Mix Script

On the channel used for X‑Any output:

```
Source: Lua
Script: xanytx.lua
```


## 5. Configure Receiver / Decoder

Ensure your X‑Any decoder is connected to the chosen RC channel.

Example with **XanySpy**:

```
PROP = 1
SW_NB = 16
```



---

# Hardware Tested

* Radiomaster **TX16S**
* EdgeTX **2.11.x**
* Multiplex **X‑Any**
* Custom Arduino **XanySpy decoder**



---

# Future Work

Planned improvements:

* Multi‑instance support (up to 4 widgets)
* Improved layout system
* Advanced slider styling
* Optional telemetry feedback



---

# Author

Original concept and testing by the project author.  
Development assistance provided via AI collaboration.

