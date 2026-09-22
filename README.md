# Adaptive Intersection Controller — Camera-Based Vehicle Detection

Camera detection zones in place of inductive loops, driving a traffic-actuated signal controller.

**To watch the live simulation, visit the project page on my site:** https://moavazpour.com/projects/adaptive-intersection-controller

## Overview

One camera per approach watches the lanes just behind the stop line. A computer-vision pipeline turns each lane into a virtual inductive loop reporting vehicle or empty, and the controller sizes every green from those signals. Deployed across three junctions with twelve cameras.

- **Company:** Anamis Electronic Vira
- **Role:** Systems & Computer Vision Engineering
- **Timeframe:** 09/2019 — 09/2022

## Problem & context

Anamis needed vehicle presence at the stop line without digging up the road for inductive loops — and without deep learning, since the edge hardware was Raspberry-Pi-class, not a GPU. Each approach's camera had to become a reliable virtual loop in real time, in daylight, glare and on wet roads where reflections look like vehicle edges, and hand its 0/1 reading to the intersection controller over a plain RS-485 serial link — the controller's only input for timing the lights.

## My role

I worked on the camera and perception side as a working student; the intersection controller's own protocol stack was built by another team — my output was just the 0/1 signal each lane sent it.

- Wrote the Python/OpenCV image-processing routines — background subtraction, edge detection and ROI-based filtering — running on Raspberry Pi-class edge hardware with IP cameras over RTSP.
- Defined a stop-line zone per lane as a virtual inductive loop, reliable for cars, motorcycles and bicycles.
- Sent each lane's 0/1 state over RS-485 to the intersection controller — its only input for signal timing across the pilot's 3 intersections and 12 cameras.

## Architecture

```
IP camera             Edge computer                0/1 per lane        Intersection
per approach  ───────► Background subtraction ───►  RS-485      ─────► controller
                        + edge detection
                        (Raspberry Pi-class)
```

## Key technical decisions

### From field problem to fix
- Counting raw edges caused false detections on wet roads, where reflections look like vehicle edges.
- Adaptive background subtraction fixed it: the system learns the empty scene and only counts what is new to it.
- The remaining errors (~5%) were all missed vehicles that had stopped outside the zone.

### How smart mode decides
- Occupancy = seconds the zones reported a vehicle ÷ green seconds given so far, both lanes weighted 50/50.
- After the minimum green, the remaining green jumps to the safety time if occupancy falls below the threshold or no vehicle passes for a set time.
- Ambulances, fire trucks and public-transport city buses request priority by V2X: their approach gets green, and the cycle resumes after they cross.

## Real deployment — Damahi intersection, Bandar Abbas

The system ran as a pilot at three intersections in Bandar Abbas, Iran — Damahi, University and Otobusrani — with 12 camera sensors. The results below were measured at Damahi intersection, where the study was carried out.

*The sites and results shown here are general, non-confidential information about a completed pilot project.*

| Metric | Value | Context |
|---|---|---|
| Cycle-time optimization based on traffic demand | 27.1% | measured during the pilot |
| Pilot duration | 79 days | continuous operation |
| Field data points | 183K+ | collected during the pilot |
| False detections | ~5% | all missed vehicles — no false alarms |

## Tech stack

Python · OpenCV · Computer vision · Background subtraction · Edge detection · ROI-based filtering · Linux · IP cameras · RTSP streaming · Camera pipelines · Embedded vision · Raspberry Pi-class edge hardware · RS-485

## Source code

This repository documents the project for portfolio purposes. The production source code itself remains proprietary to Anamis Electronic Vira — happy to walk through the architecture and trade-offs in an interview.

## License

[MIT](LICENSE) — applies to the documentation in this repository.
