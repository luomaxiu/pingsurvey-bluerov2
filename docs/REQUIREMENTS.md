# Requirements

Source: adapted from Blue Robotics' [BlueBoat bathymetry survey guide](https://bluerobotics.com/learn/collecting-creating-bathymetry-blueboat-ping2/) for a BlueROV2, which, unlike the BlueBoat, has no GPS once submerged.

Original repositories:

1. https://github.com/vshie/pingSurvey
2. https://github.com/BlueOS-community/QuickStart-Python-Extension

## Prerequisites

1. BlueROV2 equipped with a DVL and a Ping1D Echosounder.
2. Host laptop/PC with an internet connection (used only for map tile loading, the survey itself will still work when offline)

## User flow

1. On load, the user sees a map centered on: the ROV's GPS (if available) → else the laptop's internet connection (IP geolocation) → else no tile layer at all, just the ROV icon centered on a blank canvas.
2. The user inputs the GPS coordinate of the ROV's initial position while it's already in the water. This origin can be corrected retroactively in postprocessing but **not** mid-mission.
3. Before a mission plan can be set, at least two waypoints must exist. Waypoints by default are set from the DVL (`LOCAL_POSITION_NED`), with GPS being used to correct the position data when available.
4. Waypoints are set by manually navigating the ROV to each point and pressing "save" which records an (x, y, z) coordinate plus depth-sensor reading, relative to the DVL's origin. GPS is explicitly the *anchor* for that origin, not the moment-to-moment locator — DVL is. In the event of constant availability of GPS at the time of waypoint setting, GPS reading with depth sensor will be prioritized. Due to the nature of wireless communications underwater, GPS connectivity is assumed unavailable for most of the survey.
5. Two waypoints is a valid (minimum) mission — it's flown as a straight back-and-forth line, with no polygon/area pattern available.
6. Once three or more waypoints exist, the map highlights the enclosed area, even with no data in it yet.
7. That highlighted area is the survey area (QGroundControl-style).
8. The only pattern available is lawnmower, with two user-set parameters: line spacing (m) and boundary overshoot (m) — the extra distance flown past the nominal boundary on each pass to guarantee full coverage rather than undershooting.
9. No return-to-launch option for now; the ROV simply stops after the last pass Deferred to a future iteration.
10. Two automation modes: **depth-hold** (navigate to and hold a fixed depth in meters) or **contour-follow** (hold a constant distance off the seafloor using the Ping1D, only trusting readings with confidence ≥ 90%).
11. A live CSV/table view at the bottom of the canvas: DVL (x, y, z), AHRS pitch/roll/yaw/heading, Ping1D distance + confidence, GPS lat/lon, depth-sensor reading Unix epoch timestamp.
12. Using the Ping1D's known 25° beam width plus depth and distance-to-bottom, render circles on the map representing the sonar footprint at each reading.
13. The map shows the ROV's live relative position at all times.
14. The user can export the current CSV at any time.

## Non-functional requirements

- Code aligns with PEP 8.
- Project structure follows the standard BlueOS extension layout.
- Implementation proceeds in stages; passive stages (observation, logging, planning) are
  prioritized ahead of the one stage that actually drives the vehicle (mission automation),
  which is deliberately last. The extension must be fully useful under manual piloting alone —
  live relative position, logging, and export — with zero vehicle control.

## Traceability: requirement → implementation stage

| Req | Stage |
|---|---|
| Non-functional: DVL-first / GPS-anchor-only | 1 (position tracker) |
| 1 (map load fallback chain) | 1 |
| 2 (origin input, not mid-mission) | 1 |
| 11 (live CSV/table view) | 2 (logger + visualizer) |
| 12 (sonar footprint circles) | 2 |
| 13 (live relative position on map) | 2 (marker) + 1 (data) |
| 14 (export CSV any time) | 2 |
| 3, 4, 5 (waypoint fencing) | 3 (mission planner) |
| 6, 7, 8 (survey area + lawnmower pattern) | 3 |
| 9 (no return point, future feature) | 3 |
| 10 (depth-hold / contour-follow automation) | 3 |