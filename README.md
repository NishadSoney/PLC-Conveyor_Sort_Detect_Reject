# PLC-Controlled Conveyor with Sorting, Defect Detection and Rejection

A PLC-based conveyor control system that automatically detects defective objects and rejects them from the line using a pneumatic cylinder, while good items continue downstream. Programmed in Ladder Diagram on a **Mitsubishi FX5U (MELSEC iQ-F)** and verified in **GX Simulator3**.

> Third Year B.Tech Capstone Project for PLC, Electronics and Computer Science
> St. John College of Engineering & Management, University of Mumbai (2026-2027)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [I/O Assignment](#io-assignment)
- [Control Logic](#control-logic)
- [State Description](#state-description)
- [Results](#results)
- [Demo Video](#demo-video)
- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Future Scope](#future-scope)
- [Team](#team)
- [References](#references)

---

## Overview

Manual inspection of products on a conveyor is slow, inconsistent and costly when defects reach customers. This project replaces manual inspection with a compact PLC program that:

1. Runs the conveyor with a latched start/stop circuit and a start-up delay.
2. Detects a defective item and **remembers it** until it reaches the reject station.
3. Ejects the item with a pneumatic cylinder for a fixed **2-second pulse**, then retracts automatically.
4. Lets the operator pause and resume the line safely.

The entire program is only **57 steps**, which makes it a compact and readable example of sequential industrial control.

## Features

- Self-latching run circuit with Start / Stop / Enable interlock
- 10-second start-up delay timer before the belt runs
- Pause / resume latch for the conveyor drive
- Defect memory bit that travels with the item to the reject station
- Timed (2 s) reject cylinder pulse with automatic retraction
- Adjustable timer constants to handle sensor placement and belt speed
- Maximum scan time of **1.581 ms**, verified in GX Works3

## System Architecture

The FX5U PLC sits between the field inputs and outputs:

- **Operator inputs:** Start (X0), Stop (X1), Enable (X2), Pause (X6), Resume (X7)
- **Process sensors:** Gate (X3), Defect (X4), Reject station (X5)
- **FX5U PLC:** ladder program, 57 steps, 1.58 ms scan
- **Outputs:** Conveyor motor (Y0) and reject cylinder (Y1, solenoid-valve pusher)

Good items continue down the belt; a defective item is pushed off at the reject station.

## I/O Assignment

### Inputs

| Address | Function            |
|---------|---------------------|
| X0      | Start push button   |
| X1      | Stop push button    |
| X2      | Enable              |
| X3      | Gate sensor         |
| X4      | Defect sensor       |
| X5      | Reject-station sensor |
| X6      | Pause               |
| X7      | Resume              |

### Outputs

| Address | Function                          |
|---------|-----------------------------------|
| Y0      | Conveyor motor                    |
| Y1      | Reject cylinder (solenoid valve)  |

### Internal Devices

| Device | Purpose                                         |
|--------|-------------------------------------------------|
| M0     | Run latch                                       |
| M10    | Defect memory (defect pending)                  |
| M12    | Pause latch (SET by X6, RST by X7)              |
| T0     | Start-up delay, 10 s (K100, 100 ms resolution)  |
| T1     | Reject pulse, 2 s (K20, 100 ms resolution)      |

## Control Logic

1. **Run latch:** Start (X0) with Enable (X2) on and Stop (X1) off sets the latch M0.
2. **Start-up delay:** With M0 on, timer T0 counts 10 s.
3. **Conveyor run:** When T0 has elapsed and the line is not paused (M12 off), Y0 turns on.
4. **Defect memory:** A signal at X4 sets M10, which remembers the faulty item.
5. **Rejection:** When the item reaches the reject station (X5 on) and M10 is set, Y1 turns on and timer T1 starts.
6. **Retract:** When T1 elapses, M10 is reset and Y1 turns off. The cycle repeats every scan.
7. **Pause / resume:** X6 sets the pause latch M12 (conveyor off); X7 resets it.

## State Description

| State          | Condition                                         |
|----------------|---------------------------------------------------|
| Idle           | M0 off                                            |
| Starting       | Start pressed with Enable, T0 counting 10 s       |
| Running        | T0 elapsed, Y0 on                                 |
| Defect Pending | X4 detected a defect, M10 on                      |
| Ejecting       | X5 on with M10 set, Y1 on, T1 counting            |
| Paused         | X6 pressed, M12 set, Y0 off until X7 (Resume)     |

Stop (X1) returns the system to Idle from any state.

## Results

| Parameter                     | Result                  |
|-------------------------------|-------------------------|
| Program size                  | 57 steps                |
| Conversion errors             | None                    |
| Maximum scan time             | 1.581 ms                |
| Simulator status              | PWR/RUN lit, ERR off, P.RUN lit |
| Latching, timing, reject sequencing | Verified in simulation |

The logic was validated in GX Simulator3 by forcing inputs through the device test and monitoring rung states online. Hardware commissioning is the next step.

### Ladder Logic (GX Works3 Monitoring)

| Steps | Description |
|-------|-------------|
| 0-27  | Run latch M0, start-up timer T0 (K100), conveyor output Y0 with pause interlock, defect memory SET M10 |
| 19-44 | Y0 interlock, reject output Y1 with timer T1 (K20), RST M10 |
| 44-56 | RST M10, pause latch M12 (SET by X6, RST by X7) and END |

### GX Simulator3

The simulator shows the FX5UCPU in **RUN** with **PWR** and **RUN** lit and **ERR** off. The toolbar reports a maximum scan time of **1.581 ms** and the status bar shows the 57-step program.

## Demo Video

The simulation recording is uploaded in this repository in the [`media/`](media/) folder as `Simulation_Demo.mp4`. Download it or open it from the repo to watch the program run in GX Simulator3.

Diagrams, flowchart and ladder/simulator screenshots are in the [`images/`](images/) folder.

## Getting Started

### Requirements

- Mitsubishi **GX Works3** (project type: FX5U)
- **GX Simulator3** (soft PLC) for testing without hardware
- *(Optional)* Mitsubishi FX5U PLC, sensors, motor contactor, solenoid valve and pneumatic cylinder for hardware deployment

### Run in Simulation

1. Clone this repository:
   ```bash
   git clone https://github.com/NishadSoney/PLC-Conveyor_Sort_Detect_Reject.git
   ```
2. Open `PLC_Conveyor.gx3` in GX Works3.
3. Convert the program (`F4`) and confirm the Converted Result pane is empty.
4. Start **GX Simulator3** and write the program to the simulated CPU.
5. Set the CPU to **RUN**.
6. Open the device test and force inputs in this order to test the sequence:
   - Turn on **X2** (Enable), then pulse **X0** (Start). After 10 s, **Y0** turns on.
   - Pulse **X4** (Defect). **M10** is set.
   - Turn on **X5** (Reject station). **Y1** turns on for 2 s, then **M10** resets.
   - Pulse **X6** to pause and **X7** to resume.
   - Pulse **X1** to stop.

## Repository Structure

```
PLC-Conveyor-Defect-Rejection/
├── README.md                          # Project overview (this file)
├── PLC_Conveyor.gx3                   # GX Works3 project (FX5U ladder program)
├── Report.pdf                # Full capstone project report
├── images/
│   ├── Block Diagram.png              # System block diagram
│   ├── Data Flow Diagram.png          # State / data flow diagram
│   ├── Flowchart.png                  # Control flowchart
└── media/
    └── Simulation_Demo.mp4            # Simulation recording
```

| Folder | Contents |
|--------|----------|
| `images/` | Diagrams and screenshots used in the report |
| `media/`  | Simulation video |

## Future Scope

- Production counter and defect counter for quality reporting
- Shift register to track several defects on the belt at once
- Vision or colour sensors for automatic classification into several bins (true multi-way sorting)
- Emergency-stop circuit and fault alarms for the cylinder and motor
- HMI / SCADA screen with data logging over Ethernet to a database
- Variable-frequency drive for speed control and an idle-timeout energy-saving mode

## Team

| Name           | Enrollment No. |
|----------------|----------------|
| Kanchan Girase | EU1247062      |
| Aarya B. Raut  | EU2247008      |
| Dev Sehgal     | EU1247023      |
| Nishad Soney   | EU1247013      |

**Guide:** Ms. Hiral Patel, Asst. Prof., ECS, SJCEM
**Department:** Electronics and Computer Science, St. John College of Engineering & Management, University of Mumbai

## References

1. W. Bolton, *Programmable Logic Controllers*, 6th ed. Oxford, U.K.: Newnes (Elsevier), 2015.
2. F. D. Petruzella, *Programmable Logic Controllers*, 5th ed. New York, NY, USA: McGraw-Hill Education, 2016.
3. International Electrotechnical Commission, *IEC 61131-3: Programmable Controllers, Part 3: Programming Languages*, 2013.
4. Mitsubishi Electric Corporation, *MELSEC iQ-F FX5 User's Manual*.
5. Mitsubishi Electric Corporation, *GX Works3 Operating Manual*.
6. M. P. Groover, *Automation, Production Systems, and Computer-Integrated Manufacturing*, 4th ed. Upper Saddle River, NJ, USA: Pearson, 2015.

## License

Developed for academic purposes. Add a license (for example MIT) if you want others to reuse the code.
