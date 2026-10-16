# Adaptive Intersection Controller — Camera-Based Vehicle Detection

Camera detection zones in place of inductive loops, driving a traffic-actuated signal controller.

**To watch the live simulation, visit the project page on my site:** https://moavazpour.com/projects/adaptive-intersection-controller

## Overview

One camera per approach watches the lanes just behind the stop line. A computer-vision pipeline turns each lane into a virtual inductive loop reporting vehicle or empty, and the controller sizes every green from those signals. Piloted in 2021 at three intersections in Bandar Abbas with twelve cameras.

- **Company:** Anamis Electronic Vira (GambroTech)
- **Role:** Working Student – Computer Vision & Embedded Vision (part-time, remote, alongside B.Eng. studies)
- **Timeframe:** 09/2020 — 09/2022

## Problem & context

Anamis needed vehicle presence at the stop line without digging up the road for inductive loops — and without deep learning, since the edge hardware was Raspberry-Pi-class, without a GPU or neural network. Each approach's camera had to become a reliable virtual loop in real time, in daylight, glare and on wet roads where reflections look like vehicle edges, and hand its 0/1 reading to the intersection controller over a plain RS-485 serial link — the controller's only input for timing the lights.

## My role

I implemented the camera-based vehicle-presence detection as a working student, under the guidance of a senior engineer. The intersection controller's own protocol stack was owned by another team — my output was the 0/1 signal each lane sent it. The pilot results below are a team result; my part was the detection/perception module.

- Wrote the Python/OpenCV image-processing routines — adaptive background subtraction, Canny edge detection and lane-level ROIs as virtual inductive loops — running on Raspberry-Pi-class edge hardware with IP cameras over RTSP.
- Defined a stop-line zone per lane as a virtual inductive loop, covering cars, motorcycles and bicycles.
- Sent each lane's 0/1 presence state over RS-485 to the intersection controller across the pilot's 3 intersections and 12 cameras.
- Monitored the pilot remotely via logs and camera streams.
- Recovered and analysed 371,000 signal phases from a controller SD-card log corrupted by SPI noise (Python, pandas, MySQL). A 10-day vision outage served as a fixed-time comparison; the analysis also identified a heat-related availability drop and a stuck-detector fault.

## Architecture

```
IP camera             Edge computer                0/1 per lane        Intersection
per approach  ───────► Background subtraction ───►  RS-485      ─────► controller
                        + edge detection
                        (Raspberry Pi-class)
```

## Key technical decisions

### From field problem to fix
- Counting raw edges caused false positives on wet asphalt, where reflections look like vehicle edges.
- Adaptive background subtraction fixed it: the system learns the empty scene and only counts what is new to it. This effectively eliminated false positives.
- The remaining miss rate of ~6% came from vehicles that stopped outside the detection zone.

### How smart mode decides
- Occupancy = seconds the zones reported a vehicle ÷ green seconds given so far, both lanes weighted 50/50.
- After the minimum green, the remaining green jumps to the safety time if occupancy falls below the threshold or no vehicle passes for a set time.
- Ambulances, fire trucks and public-transport city buses request priority by V2X: their approach gets green, and the cycle resumes after they cross.

### Fail-safe
- The junction ran on fixed time through a 10-day vision outage and returned to adaptive mode automatically.

## Real deployment — Damahi intersection, Bandar Abbas (2021)

The system ran as a pilot at three intersections in Bandar Abbas, Iran — Damahi, University and Otobusrani — with 12 camera sensors. The results below were measured at Damahi intersection, where the study was carried out (team result).

*The sites and results shown here are general, non-confidential information about a completed pilot project.*

| Metric | Value | Context |
|---|---|---|
| Logged adaptive operation | 123 days | at the evaluated junction |
| Adaptive-mode availability | 92.5% | in heat up to 45 °C, excluding a 10-day hardware outage |
| Average cycle time | 12% shorter | vs. the same junction's fixed-time plan in the same season (64.7 s → 56.9 s) |
| Cycle time vs. original municipal plan | ~20% shorter | 71 s → 56.9 s |
| Cycles per day | +14% | |
| Main-road green share | 49% → 57% | |
| Main-road wait at red | ~28% lower | 10.5 s → 7.6 s, random-arrival estimate |
| Evening-peak adaptation | up to +14% | cycle lengthened automatically at the evening peak |
| Signal phases analysed | 371K | recovered from a corrupted controller SD-card log |
| Detection miss rate | ~6% | vehicles stopping outside the ROI; false positives effectively eliminated |

The pilot was formally commended by the Bandar Abbas traffic department, which requested an extension to more intersections.

## Tech stack

Python · OpenCV · Computer vision · Adaptive background subtraction · Canny edge detection · ROI-based filtering · pandas · MySQL · Linux · IP cameras · RTSP streaming · Embedded vision · Raspberry Pi-class edge hardware · RS-485

## Source code

This repository documents the project for portfolio purposes. The production source code itself remains proprietary to Anamis Electronic Vira — happy to walk through the architecture and trade-offs in an interview.

## License

[MIT](LICENSE) — applies to the documentation in this repository.
