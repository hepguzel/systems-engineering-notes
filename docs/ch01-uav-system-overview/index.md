# Chapter 1 — UAV System Overview

This chapter introduces the operational lifecycle of a single-use, fixed-wing expendable UAV platform. The mission profile is examined from initial power application through terminal phase and end-of-mission. Understanding this lifecycle is essential before defining system architecture, subsystem interfaces, or requirements, as each mission phase imposes distinct functional, safety, and performance constraints on the overall system design.

---

## 1.1 Mission Profile

Before defining architecture or requirements, we must understand the mission profile.

A mission profile is a time-ordered sequence of operational phases that describes what the UAV does from the beginning of its operation until the end of its life.

It defines:

- Which subsystems must exist  
- What sensors are required  
- What performance levels are necessary  
- How safety logic behaves  
- What requirements must be written  
- How the architecture must support each phase  

For a single-use expendable UAV, the mission is typically short-duration, pre-planned, fully autonomous, low-cost, non-recoverable, and designed for terminal destruction or impact.

### Complete Mission Phases Overview

For this platform, the mission consists of the following phases:

- Power-On and Initialization  
- Built-In Test (BIT)  
- Mission Upload  
- Pre-Arm Checks  
- Arming  
- Launch  
- Autonomous Flight  
- Terminal Phase  
- End-of-Mission  

Each phase activates different subsystems and introduces different risks. System architecture must support all phases coherently.

---

### 1. Power-On and Initialization

The mission begins when electrical power is applied to the UAV.

This typically occurs when:

- The battery is connected  
- External ground power is applied  

At this moment:

- The flight control computer boots  
- Firmware starts executing  
- Configuration parameters are loaded  
- All connected sensors begin initialization  

Typical sensor startup actions include:

- IMU initialization  
- Gyroscope bias estimation  
- Accelerometer offset correction  
- Gravity vector detection  
- Barometer stabilization  
- GNSS satellite acquisition  
- Air data sensor zeroing (if present)  
- Servo output checks  
- ESC or engine controller self-test  

This phase ensures that the system transitions from a powered but inactive state to an operational standby state.

---

### 2. Built-In Test (BIT)

During startup, the UAV executes Built-In Tests.

The objective of BIT is to verify that:

- All subsystems are communicating correctly  
- No hardware faults are detected  
- Sensor values are within expected ranges  
- No critical flags are raised  

If any major failure is detected:

- The system prevents progression to arming  
- The operator is notified  
- The mission cannot proceed  

BIT is the first architectural safety gate. Every subsystem has an interface and expected behavior; by this means you verify that all subsystems report “healthy.”

---

### 3. Mission Upload

After successful initialization, the mission plan is uploaded from the Ground Control Station (GCS).

The mission data typically includes:

- Waypoints (latitude, longitude, altitude)  
- Speed commands (climb, cruise, terminal speed)  
- Behavior definitions (impact, climb-dive, fast pass)  
- Fail-safe logic (loss-of-link timeout, hold/continue logic, termination rules)  
- Special mission parameters (e.g., radar cross-section pattern, specific maneuver timing, altitude constraints)  

Example mission sequence:

- WP1: Climb to 200 m  
- WP2: Cruise segment  
- WP3: Turn point  
- WP4: Terminal coordinate (impact point)  

After receiving the mission, the autopilot:

- Saves the mission  
- Checks whether the mission is valid  
- Computes the expected flight path  
- Validates safe minimum altitudes  
- Aligns required launch direction  
- Pre-calculates where to turn  
- Calculates estimated time to reach terminal point  

If any validation fails:

- The UAV refuses to arm  
- The operator must correct the mission  

---

### 4. Pre-Arm Checks

Before transitioning to an active state, the UAV performs automatic pre-arm checks.

Typical conditions include:

- GNSS position lock achieved  
- IMU attitude stable  
- No active fault flags  
- Power system voltage within limits  
- No sensor timeout conditions  
- Vehicle stationary (if required)  

If any condition fails:

- ARMED state remains FALSE  
- Propulsion is inhibited  
- Operator receives warning  

Pre-arm checks prevent unsafe launch attempts.

---

### 5. Arming Phase

The Arming Phase is a safety-critical transition.

Arming is defined as the transition from a safe, inhibited configuration to an operational configuration in which propulsion and control outputs are enabled.

When `ARMED = TRUE`:

- Throttle commands activate propulsion  
- Control surfaces respond fully  
- Flight termination logic becomes active  
- The system is ready for launch  

When `ARMED = FALSE`:

- Propulsion is disabled  
- Control outputs may be limited  
- Certain subsystems remain inhibited  

Arming may require:

- Operator command (via GCS or hardware switch)  
- Physical safety plug removal  
- Confirmation of pre-arm criteria  

Arming represents a formal architectural boundary between the ground-safe state and the flight-ready state.

In system engineering terms, it is a controlled state transition with safety interlocks.

---

### 6. Launch Phase

Launch is the moment when the UAV transitions from a static object to a flying aircraft.

Launch is the most stressful part of the mission because:

- High acceleration occurs  
- Airflow becomes chaotic  
- Aerodynamic forces build rapidly  
- Control surfaces may initially be ineffective  
- IMU experiences shock loading  
- GNSS signal may temporarily degrade  
- Autopilot must switch flight modes at the correct moment  
- Thrust becomes required immediately  

For expendable UAVs, common launch methods include:

- Catapult launch  
- Rocket-assisted launch  
- Runway takeoff  
- Vertical launch  

**Catapult Launch**

Catapult launch systems can be elastic (rubber band sling), pneumatic (air pressure piston), or rail launcher with bungee assist. The UAV typically accelerates between 5–15 g (sometimes up to 20 g), and this acceleration phase lasts approximately 0.3–1.0 seconds. The flight controller operates in a dedicated “launch mode,” in which control surfaces may be fixed, the autopilot temporarily limits active corrections, and sensors anticipate shock loading. The UAV detects launch through acceleration above a threshold, rising airspeed, and rail switch logic. The motor or engine provides full power, the IMU experiences high shock, and the UAV leaves the rail.

**Rocket-Assisted Launch (RAL)**

Rocket-Assisted Launch is used for heavier expendable drones where the electric motor is not sufficient, the catapult length is too short, or an immediate steep climb is required. The UAV is placed on a rail, the booster ignites, strong initial thrust is generated, the UAV accelerates rapidly, the booster burns out and separates, and the main propulsion system takes over.

**Vertical Launch**

In a vertical launch configuration, the UAV lifts off vertically using dedicated lift motors or a combined propulsion system capable of generating upward thrust. The vehicle climbs straight up until it reaches a safe altitude, then transitions into forward flight. The most critical challenge in vertical launch is the transition phase, when the aircraft shifts from vertical lift to aerodynamic flight. During this transition, control logic must reconfigure smoothly while aerodynamic surfaces gradually become effective.

**Runway Takeoff**

In runway takeoff, the UAV accelerates along a prepared surface until sufficient airspeed is reached for lift-off. Acceleration is gradual and controlled, and aerodynamic surfaces become effective progressively as airspeed increases. Because there is no sudden shock or extreme acceleration, IMU measurements remain stable and navigation filters operate under normal conditions. The key requirement is sufficient runway length and reliable propulsion performance.

---

### 7. Autonomous Flight Phase

After achieving safe airspeed and stable attitude, the UAV enters autonomous flight.

During this phase, the system:

- Keeps itself stable  
- Follows mission plan  
- Navigates between waypoints  
- Manages altitude, speed, and thermal behavior  
- Reacts to wind and disturbances  
- Monitors all subsystems  
- Executes fail-safes  

Navigation between waypoints is typically performed using guidance algorithms such as L1 navigation. L1 navigation is used by ArduPilot, PX4, many commercial autopilots, and several proprietary autopilot systems. It creates a smooth, curved flight path instead of sharp angular turns.

The autopilot continuously:

- Estimates current position  
- Computes heading to next waypoint  
- Adjusts roll, pitch, and throttle  
- Switches to next waypoint upon reaching acceptance radius  

This phase represents steady-state mission execution.

---

### 8. Terminal Phase

The terminal phase depends on mission design.

Possible terminal behaviors include:

- Direct impact into target coordinate  
- Climb-and-dive profile  
- Controlled crash in predefined safe zone  
- Flight Termination System (FTS) activation  

In expendable systems, recovery is not expected.

Terminal logic must be deterministic and predictable.

---

### 9. End-of-Mission

For a single-use expendable UAV:

- There is no recovery  
- The vehicle is intentionally destroyed or impacted  
- Mission ends upon impact or termination  

System requirements must clearly define:

- Acceptable end-of-life conditions  
- Safety perimeter  
- Failure contingencies  

---

## 1.2 System Boundaries

The system boundary of a UAV defines the separation between internal system elements and external actors.

The system boundary defines:

- What is inside the UAV system (elements you design, control, and are responsible for)  
- What is outside the system (external actors and systems the UAV interacts with but does not design or physically contain)  

System boundaries help you:

- Identify what you design and clarify what you are responsible for  
- Limit scope creep  
- Structure SRS and architecture documents  
- Coordinate with cross-functional teams  
- Define what belongs in the SRS or the IDS (Interface Definition Specification)  

### Inside the System Boundary

1. **Air Vehicle**
   
   - Airframe & Structures  
   - Propulsion System (motor/engine, ESC, fuel system)  
   - Avionics (flight controller, GNSS, IMU, magnetometer, air data sensors)  
   - Electrical power system (battery, PDB/regulators, wiring harness)  
   - Control actuators (servos)  
   - Payload (camera or mission module)  

2. **Embedded Software**
   
   - Flight control firmware  
   - Autopilot modes  
   - Safety logic (fail-safes, loss of link)  

3. **Internal Data & Power Interfaces**
   
   - All internal power, data, control lines, and mechanical mounts  

### Outside the System Boundary

The outside includes everything the UAV interacts with but does not design or physically contain.

1. **Ground Control Station (GCS)**
   
   - Laptop or tablet  
   - GCS software  
   - Telemetry radios  

2. **Operators**
   - UAV pilot/operator  
   - Launch crew  
   - Mission commander  

3. **Launch Equipment**
   
   - Catapult  
   - Rocket-assisted launcher  
   - Runway or airfield  
   - Vertical launch stand  

4. **Environment**
   
   - Atmosphere (wind, temperature, pressure)  
   - Terrain  
   - GNSS satellite constellation  
