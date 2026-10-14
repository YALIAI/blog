# CARLA Autonomous Driving Control: Quartic Speed Planning, PID and Pure Pursuit

> **Project status (2026-10-14): Work in progress.** The simulator and control pipeline have been implemented; the latest quartic braking changes still require end-to-end validation in CARLA. Demo video: **to be added**.

## Goal

Build a reproducible autonomous-driving control demonstration in **CARLA 0.9.16** using a built-in map (**Town04**) with the following driving sequence:

1. Accelerate from 0 to **50 km/h** (13.89 m/s).
2. Cruise at the target speed.
3. Decelerate smoothly and stop near the route endpoint.
4. Track the route laterally and visualize the vehicle's deviation from the lane/path center.
5. Record a camera/HUD video with reference and actual speed traces.

The Göteborg/Marklandsgatan OpenStreetMap → OpenDRIVE experiment is tracked separately; this control demonstration currently uses Town04 to avoid issues with the imported road network.

## Architecture

| Component | Role |
|---|---|
| `main.py` | CARLA connection, map loading, route selection, simulation loop, driving phases and stop logic |
| `trajectory.py` | Quartic speed reference and signed lateral error |
| `controller.py` | PID longitudinal controller and Pure Pursuit lateral controller |
| `hud.py` | RGB camera view, driving metrics, real-time speed traces and MP4 recording |

The closed loop is: **route / speed planner → PID throttle-brake + Pure Pursuit steering → CARLA vehicle → measured state feedback**.

## 1. Quartic speed reference

For each acceleration or deceleration segment, define normalized time `s = clamp(t/T, 0, 1)`:

```python
v_ref = v0 + (v1 - v0) * (4*s**3 - 3*s**4)
```

This is a fourth-order polynomial in time. It satisfies `v(0)=v0`, `v(T)=v1`, and zero **first derivative of velocity** (acceleration) at both endpoints. It does **not** guarantee zero jerk at both endpoints or C² continuity when arbitrary segments are joined.

The latest planner organizes the run into **ACCEL → CRUISE → BRAKE**, with an additional distance-based safety constraint when necessary. As a result, the commanded reference may deviate from the pure quartic curve when safety limits intervene.

### Correct quartic braking distance

For `v1=0`:

```text
v(s) = v0 * (1 - 4*s^3 + 3*s^4)
D = integral(v dt) = 0.4 * v0 * T
```

An earlier implementation incorrectly used **`0.8 * v0 * T`**, overestimating braking distance by a factor of two and potentially triggering braking too early. This was corrected in a subsequent version. A low-speed recovery mechanism was also introduced for the case where the time-based braking trajectory ends before the vehicle reaches the spatial destination.

**Important:** This distance is a *reference trajectory integral*, not a guaranteed physical stopping distance. Vehicle dynamics, controller tracking error and any safety override affect the actual stopping point.

## 2. Longitudinal control: PID

The longitudinal controller tracks reference speed using proportional, integral and derivative terms, translating its output to **throttle** or **brake**. The implementation was revised to mitigate integral windup and prevent unwanted throttle during braking. Stop-hold logic was added near the destination.

A diagnostic log exposes actual speed, reference speed, distance-based desired speed, remaining route distance, phase, lateral error, steering, throttle and brake.

## 3. Lateral control: Pure Pursuit

Pure Pursuit chooses a lookahead point on the route, transforms it into the vehicle frame and computes a steering command from the target angle and wheelbase. The lookahead distance grows with vehicle speed.

Signed lateral deviation is displayed for the vehicle reference point relative to the **planned route**. The current CoG is an approximation based on the vehicle actor and bounding-box offset, **not** a calibrated physical center of gravity.

## 4. Visualization and recording

The Pygame HUD integrates:

- CARLA RGB camera feed.
- Speed, steering, throttle, brake and signed lateral deviation.
- **Reference Speed vs Actual Speed** rolling graph (km/h).
- MP4 recording of the combined view.

**Demo video:** _Pending upload; insert video or link here._

## 5. Observations and debugging history

Earlier logs showed an actual speed of approximately **13.19 m/s (47.5 km/h)** in one captured segment. A separate stationary log showed lateral deviation around **−0.03 m**, which describes only that instant, not full-route tracking accuracy.

Several issues were observed and investigated:

- **Throttle remained positive during deceleration:** possible PID integral accumulation; controller revised.
- **Lateral error sign jumped abruptly:** nearest-lane projection could switch between lane centers; route-relative error adopted.
- **Stopped 3.1 m short and loop did not terminate:** stop completion tolerance and hold logic revised.
- **Stopped about 45.2 m short while BRAKE continued:** the time-based speed reference reached zero before the remaining route distance; investigation identified the erroneous quartic distance coefficient above. The latest correction has **not yet been confirmed by a new driving log**.

These observations should not be treated as proof of stable 50 km/h tracking or successful quartic braking.

## 6. Validation plan

- Verify complete **0 → 50 → 0 km/h** operation and stopping position on Town04.
- Plot the reference and measured speeds for the full run.
- Calculate speed tracking MAE/RMSE and maximum error.
- Calculate lateral deviation RMSE and maximum absolute deviation.
- Inspect camera video for collisions, lane departures and braking behavior.
- Add an annotated demo video and tested configuration after validation.

## 7. Separate experiment: Göteborg OpenDRIVE

An OpenStreetMap extract around Marklandsgatan, Göteborg, was converted using CARLA's OSM-to-OpenDRIVE workflow. The imported map provided road geometry but lacked the full buildings/vegetation visual environment. Work on the Göteborg map is paused while the built-in-map control experiment is stabilized.

---

**Stack:** CARLA 0.9.16 · Python · Pygame · PID · Pure Pursuit · quartic polynomial velocity planning · OpenDRIVE.
