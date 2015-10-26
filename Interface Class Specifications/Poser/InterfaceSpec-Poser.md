# Interface Class Description: Poser

> Status: **partially implemented** - legacy available through VRPN.

## Device class summary
This is a general "inverse" of the tracker device interface class: it allows a client to command a pose, velocity, acceleration, etc. subject to the limitations of the hardware. As with the tracker, not all devices actuate all degrees of freedom or provide access to actuating all derivatives.

This class is one of the VR device classes carried over from VRPN, though less commonly used than the classic classes like tracker and button, and some choices and practices are influenced by VRPN and other existing software. To keep a consistent system, the units, reference frames, etc. for this class parallel those found in Tracker.

A poser device may expose one or more "sensors" (in this case, actuated bodies). Not all sensors may provide the same degrees of actuation, but unlike trackers, the number of "sensors" is typically bounded and known.

### Examples
- IMU-based orientation trackers integrated into head-mounted displays
- Magnetic, optical, and other 6DOF VR trackers
- Mechanically-linked orientation and height trackers included on some kinds of navigation devices
- Simulated trackers used as a means of representing a user's location in a virtual environment to provide for exploration.

See the extensive list of hardware, primarily trackers, (legacy and contemporary) supported by VRPN: <https://github.com/vrpn/vrpn/wiki/Supported-hardware-devices> for specific examples.


### Relation to other classes
**Factoring**: While each component of the report is representable as a floating-point value, the specific nature of data as pose-related indicates that such data should be reported as tracker data whenever possible. Some commercial devices integrate tracking and gamepad-like controls: these are typically exposed as tracker interfaces with associated button and analog interfaces. The descriptor should be used to make the semantic association of particular buttons/analogs with specific tracking sensors.

**From other instances of the same interface**: Sensor fusion, filtering, and predictive tracking algorithms are logical analysis possibilities that would both take in tracker input and present a tracker interface.

**From other classes**: VRPN provides "AnalogFly" and "ButtonFly" simulated trackers that allow use of analog devices (joysticks, SpaceMouse-type devices, etc) and button devices (gamepad D-Pads, etc) to simulate trackers. Other sensing devices (cameras, etc) might also be analyzed by some algorithm to produce tracking data.

## Overview
The Poser interface is summarized in the following diagram:

![Poser interface class](PoserInterface.png)

## Messages
When tracker messages reach the application, they should be in the global coordinate system. If some other coordinate  system applies, a default transformation should be supplied in the JSON descriptor, and/or configured (for instance, if a tracker origin is attached to another tracked object such as an HMD).

Devices should only report what they observe, at least on their primary device name: there is no need to compute the derivatives, or add pose integration in the driver. (Of course, if a device embeds sensor fusion, reporting that output is proper.) Additionally, each sensor should only report one of the message types for each "level" (pose, velocity, acceleration): the message that is the closest match to the device observation. 

Note that becaused of the VRPN core, the so-called "subset" messages (reporting less than 6DOF) are embedded in a full 6DOF VRPN message (with missing components zero or identity) on the wire for backward compatibility. This does not affect native OSVR clients or OSVR game engine integrations, as the client library has the descriptor data to determine what portions of the message are real data.

With respect to subset messages, this document is written with the device driver author primarily in mind. Client applications can access full or subset data as available, and full reports also trigger subset callbacks/populate subset data.

Each message reports data on only a single sensor, and (in part because the set of sensor IDs is logically unbounded) no initial state is automatically transmitted to new clients outside of normal device operation.

### Pose
#### Data
- Sensor number
- Position (3D vector)
- Orientation (unit/normalized quaternion)

#### Rationale
This is the full, basic tracker message: providing a coordinate system at a sensor.

### Position
This is a subset message of **Pose**, for sensors that report only position.

#### Data
- Sensor number
- Position (3D vector)

### Orientation
This is a subset message of **Pose**, for sensors that report only orientation. If a sensor sends a **Pose** message, there is no need to send a separate **Orientation** message for that same observation.

#### Data
- Sensor number
- Orientation (unit/normalized quaternion)

---

### Velocity
#### Data
- Sensor number
- Linear velocity (m/s) (3D vector)
- Incremental rotation per delta time (unit/normalized quaternion)
- Delta time for incremental rotation (seconds)

#### Rationale
This is the full "first derivative" tracker message. It, and all of its subset messages, are in the same coordinate system as the Pose messages (that is, the global coordinate system).

### LinearVelocity
This is a subset message of **Velocity**, for sensors that report only linear/translational velocity.

#### Data
- Sensor number
- Linear velocity (m/s) (3D vector)

### AngularVelocity
This is a subset message of **Velocity**, for sensors that report only rotational velocity (such as rate gyros).

#### Data
- Sensor number
- Incremental rotation per delta time (unit/normalized quaternion)
- Delta time for incremental rotation (seconds)

---

### Acceleration
#### Data
- Sensor number
- Linear acceleration, exclusive of gravity (m/s^2) (3D vector)
- Increment in incremental rotation per delta time (unit/normalized quaternion)
- Delta time for rotation (seconds)

#### Rationale
This is the full "second derivative" tracker message. It, and all of its subset messages, are in the same coordinate system as the Pose messages (that is, the global coordinate system).

Note that gravity is not included in the linear acceleration component: a sensor that is fully still should report 0 acceleration.

### LinearAcceleration
This is a subset message of **Acceleration**, for sensors that report only linear/translational acceleration, such as an accelerometer (though accelerometer data should have the effect of gravity filtered out of it first).

#### Data
- Sensor number
- Linear acceleration, exclusive of gravity (m/s^2) (3D vector)

### AngularAcceleration
This is a subset message of **Acceleration**, for sensors that report only rotational acceleration.

#### Data
- Sensor number
- Increment in incremental rotation per delta time (unit/normalized quaternion)
- Delta time for rotation (seconds)


## Open issues

- The relationship between trackers and locomotion devices is somewhat open. Present thinking is that most consumer-related locomotion devices provide a locomotion interface (which is primarily a 2D movement - velocity on the plane) as well as a limited tracker interface that reports the direction a user is facing and height for crouching/jumping.
- The current derivative messages can provide raw data from only two of the three instruments integrated into modern "9-axis" IMUs: rate gyroscopes and accelerometers can be reported in the derivative messages, but the magnetometer/compass data cannot. (And, the direction of gravity detected by the accelerometer cannot be reported.) Presently, compass data is either reported in a vendor-specific way in analog channels or not reported at all, but to permit sensor-fusion algorithms within OSVR it might be useful to have a standardized compass report.
- A number of predictive tracking and sensor fusion algorithms (the family derived from the Kalman filter, most recognizably) operate using the data and a measurement of uncertainty. There is currently no standardized way to report uncertainty in measurements in this (or other) interfaces, besides an analog channel with user-defined semantics or fixed uncertainty reported in JSON descriptions.
- Might a single sensor conceivably report just a position at one time and at some later time report orientation only or a full pose? Or, can we infer that the particular subset/full message sent is the data sent on that channel?
- The preferred semantics for handling "downgrading" of messages to subset messages is currently an open topic.
