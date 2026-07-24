# Implementation Plan

Three stages, ordered by risk: everything through Stage 2 is purely observational (reads telemetry, writes logs, renders a map) and works standalone under manual piloting. Stage 3 is the only one that sends the vehicle anywhere, and is built last.

## Stage 1 - Scaffolding + Position Tracker

**Goal:** A running extension that knows where the ROV is, without yet doing anything visible with that information beyond exposing it over the API.

## Stage 2 - Logger + Visualizer

**Goal:** Everything the spec needs for a fully manual, fully observed survey dive — nothing here drives the vehicle.

## Stage 3 - Mission Planner

**Goal:** Everything in Reqs 3–10 — waypoint fencing, lawnmower pattern generation, and the two automation modes. This is the only stage that issues motion commands to the vehicle, so it's sequenced last and will get the most deliberate treatment (see safety notes below).