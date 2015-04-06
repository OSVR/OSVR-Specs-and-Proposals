# Interface Class Description: Tracker

## Device class summary
This is a basic device class used to expose data that is some way a position, orientation, both, or their derivatives (velocity, acceleration). Note that in this document and elsewhere, *pose* is a term used to mean "position and orientation". Not all devices can report a full 6 degree of freedom pose (6DOF - 3DOF in translation/position, 3DOF in rotation/orientation), and many devices do not report any derivatives (for example). This wide variation in data reporting is accommodated in two ways:

- A number of message types are provided, and some specific "subset" messages are defined (orientation only, for example)
- The JSON self-descriptor can document further any information on the device.

This class is one of the "classic" VR device classes, and some choices and practices are influenced by VRPN and other existing tracker-interface software. As it describes spatial information, and the reference frames can vary between different sources of data, this is historically one of the more difficult aspects of a VR system to configure. We aim to reduce that difficulty by standardizing on a world coordinate system and units, as well as providing methods in the system for devices to describe their relationship to one another and for these relationships to be described by external code. While not all manual configuration can be eliminated in complex VR systems, simple VR systems such as those considered for consumer virtual reality gaming should be autoconfigurable.

A tracker device may expose one or more sensors. Not all sensors may provide the same information, and the number of sensors may be unbounded: for instance, the ART DTrack2 system allows 6DOF tracking of (bounded) registered constellations of optical markers, as well as tracking of an unbounded (over time) number of single optical markers in 3DOF.

Somewhat uniquely among the interface classes, Tracker interfaces have built-in "transformation" support in the core without analysis plugins required. This is because the basic use case for these devices requires transformations to arrive at internally consistent data.

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

## Messages
When tracker messages reach the application, they should be in the global coordinate system. If some other coordinate  system applies, a default transformation should be supplied in the JSON descriptor, and/or configured (for instance, if a tracker origin is attached to another tracked object such as an HMD).

Devices should only report what they observe, at least on their primary device name: there is no need to compute the derivatives, or add pose integration in the driver. (Of course, if a device embeds sensor fusion, reporting that output is proper.) Additionally, each sensor should only report one of the message types for each "level" (pose, velocity, acceleration): the message that is the closest match to the device observation. 

Note that because of the VRPN core, the so-called "subset" messages (reporting less than 6DOF) are embedded in a full 6DOF VRPN message (with missing components zero or identity) on the wire for backward compatibility. This does not affect native OSVR clients or OSVR game engine integrations, as the client library has the descriptor data to determine what portions of the message are real data.

With respect to subset messages, this document is written with the device driver author primarily in mind. Client applications can access full or subset data as available, and full reports also trigger subset callbacks/populate subset data.

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
This is the full "first derivative" tracker message.

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
This is the full "second derivative" tracker message. Note that gravity is not included in the linear acceleration component: a sensor that is fully still should report 0 acceleration.

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

- What coordinate system are the velocity and acceleration in? Global coordinate system would make the most sense.
- The relationship between trackers and locomotion devices is somewhat open. Present thinking is that most consumer-related locomotion devices provide a locomotion interface (which is primarily a 2D movement - velocity on the plane) as well as a limited tracker interface that reports the direction a user is facing and height for crouching/jumping.
- The current derivative messages can provide raw data from only two of the three instruments integrated into modern "9-axis" IMUs: rate gyroscopes and accelerometers can be reported in the derivative messages, but the magnetometer/compass data cannot. (And, the direction of gravity detected by the accelerometer cannot be reported.) Presently, compass data is either reported in a vendor-specific way in analog channels or not reported at all, but to permit sensor-fusion algorithms within OSVR it might be useful to have a standardized compass report.
- A number of predictive tracking and sensor fusion algorithms (the family derived from the Kalman filter, most recognizably) operate using the data and a measurement of uncertainty. There is currently no standardized way to report uncertainty in measurements in this (or other) interfaces, besides an analog channel with user-defined semantics.
- Might a single sensor conceivably report just a position at one time and at some later time report orientation only or a full pose? Or, can we infer that the particular subset/full message sent is the data sent on that channel?
- The preferred semantics for handling "downgrading" of messages to subset messages is currently an open topic.

## Other resources
- TODO