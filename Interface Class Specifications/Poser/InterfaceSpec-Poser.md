# Interface Class Description: Poser

> Status: **partially implemented** - legacy available through VRPN.

## Device class summary
This is a general "inverse" of the tracker device interface class: it allows a client to command a pose, velocity, acceleration, etc. subject to the limitations of the hardware. As with the tracker, not all devices actuate all degrees of freedom or provide access to actuating all derivatives.

This class is one of the VR device classes carried over from VRPN, though less commonly used than the classic classes like tracker and button, and some choices and practices are influenced by VRPN and other existing software. To keep a consistent system, the units, reference frames, etc. for this class parallel those found in Tracker.

A poser device may expose one or more "sensors" (in this case, actuated bodies). Not all sensors may provide the same degrees of actuation, but unlike trackers, the number of "sensors" is typically bounded and known.

Note that for a variety of reasons (safety, physical limits of the device, etc.) a request from a client for the device to be actuated to a given pose, velocity, or acceleration is only a request, not a guarantee that the request will be honored within a given timeframe or at all. (Devices are free to use whatever control scheme, etc. desired.) For situations where the results of a request matter to an application, a matching Tracker interface sensor corresponding to the actual state of the device is suggested. If the device does not contain encoders sufficient to report this information, an external tracker may be used for this purpose.

As this is an output device, only one application using such a device should be run at a single time. Operation when two clients are actively sending requests to the same poser interface "sensor" is officially undefined. Client requests go directly to the server and are not broadcast to other clients.

### Examples
- Motion-base chairs
- Mixed-reality props (movable walls and other scenery, etc.)

### Relation to other classes
**Factoring**: This should be used any time the data flow goes from client application to device and the meaning is some subset of pose, velocity, or acceleration. (The VRPN Analog_Output interface is most similar, but is for non-pose-related output.)

**Association with other classes**: When reasonable, a corresponding sensor on a Tracker interface should be provided to permit observation of the current state of the device by one or more clients. This information may be used by configuration or an analysis device to "correct" data reported by other tracker sensors, for instances where the motion affects other tracked objects but such change should be disregarded (for example, motion-simulation bases).

**From other classes**: Devices may choose to internally use Analog Output to control actuators to produce the desired output, though such a transformation is linked to a given device's structure and design, and not likely to be generic.

## Messages
When poser API calls are performed by the application, they are typically in the room coordinate system. OSVR provides for configurable spatial transformations so that the actual messages sent over the transport and received by the device plugin are in whatever system the device expects. For simplicity, the device should align its coordinate system with the room coordinate system where possible.

### Pose
#### Data
- Sensor number
- Position (3D vector)
- Orientation (unit/normalized quaternion)

#### Rationale
This is the full, basic poser message: providing a desired coordinate system at a sensor.

---

### Velocity
#### Data
- Sensor number
- Linear velocity (m/s) (3D vector)
- Incremental rotation per delta time (unit/normalized quaternion)
- Delta time for incremental rotation (seconds)

#### Rationale
This is the full "first derivative" poser message. It is in the same coordinate system as the Pose messages (that is, the room coordinate system) at the client API level and subject to the same transformations.

---

### Acceleration
#### Data
- Sensor number
- Linear acceleration, exclusive of gravity (m/s^2) (3D vector)
- Increment in incremental rotation per delta time (unit/normalized quaternion)
- Delta time for rotation (seconds)

#### Rationale
This is the full "second derivative" poser message. It is in the same coordinate system as the Pose messages (that is, the room coordinate system) at the client API level and subject to the same transformations.

As with the tracker interface, linear acceleration requests should not include the effects of gravity: if a client wants to change the direction of gravity, a pose request is the way to do that.

---

## Open issues

- Trackers report their measured degrees of freedom in device descriptor data, which flows from server to client, The existing Poser reports are full reports, with no field to indicate the activity/validity of a component (to easily send subset reports). How to manage requesting a subset of the full pose/velocity/acceleration is still open.