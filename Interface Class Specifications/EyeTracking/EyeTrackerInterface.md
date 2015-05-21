# Interface Class Description: Eye Tracker

> status: in discussions

## Russ comments
I have a rather different viewpoint on how to factor the eye-tracker devices, which I'll add as a separate section here because I don't want to remove the original viewpoint described below.  The discussion below seems to be saying "what multiple fields would we want to add to support a set of devices with different functions, where some functions are not present on all devices?".  Within VRPN, we faced this issue with hand-held tracking devices, some of which had buttons and others of which had analogs, and some (like the Phantom) had force displays.  The solution was to factor the "device driver" into various interfaces (as you well know, since the other part of this repository is on factoring).

It seems to me that the discussion below has not completely finished factoring the device, so there are still optional features that not all will support.  After reading the proposal, I would tend to factor the devices something like this:
- 3D gaze direction would be a new interface/feature that is a unit directional vector in 3D with no particular origin (vrpn_Direction).  Other instances of this interface could include the gravity direction and the "north" direction from inertial tracking units.  For a monocular unit, there would be one such interface; for a two-eyed unit, there would be two.  (VRPN is not very good at figuring out how to handle sets of nested structures, so this would probably be done by having the direction-vector interface supporting one or more elements, like the tracker has sensors or button device has button counts).
- 2D gaze direction as a 0-1 sounds like a location within a 2D region.  Another example of this would be a finger location on a touch-screen display (maybe we call it vrpn_2DLocation). We're going to have to be careful to think about how to map these to pixel coordinates (and whether to handle out-of-range values).  For example, (0,0) may be the outer edge of pixel (0,0) on a screen and (1,1) may be the far outer edge of pixel (1279, 1079), so the lower-left corner of the last pixel would be (1279/1280, 1079/1080) and the middle of the lower-left pixel would be (0.5/???, 0.5/???).  There would be one for each eye to the report list for an eye tracker that supports this.  We should define the coordinate system as right handed with lower-left = origin.
- Blink and other events seem either like buttons (start of blink/end of blink, start of saccade/end of saccade, start of fixation/end of fixation) or like the more general named-event system we've been talking about for OSVR.  If named events, then they would each be a new interface/feature type and all of them will have a boolean to report state (on the wire, they will be the same as buttons, but semantically they will be distinguishable).  Within the current OSVR framework, I would tend to make them all button events and then have the semantics baked into the Json files that describes/names them.  This would let you hook them up to arbitrary button-based triggers in an application.  Each device that has blinks will have one or two buttons that correspond to blinking, one for each eye.  Same for saccade and fixation.

I see the appeal in trying to nest the objects as binocular { monocular left, monocular right }, but doing this at the data level rapidly gets us into the soup of a different driver for each device: phantom { threevec force, threevec position, fourvec orientation, button buttons[] }.  I really like the idea of keeping the data as raw as possible but building the semantics into the Json file, which tells which button corresponds to the left-eye blink and so forth.  I think this gives us the best of both worlds -- orthogonal decomposition of data and semantic hierarchy.

    OSVR_ETrackerInstance1 : public vrpn_Direction, vrpn_2DLocation, public vrpn_Analog, public vrpn_Button;
    OSVR_ETrackerInstance2 : public vrpn_Direction, vrpn_2DLocation, public vrpn_Analog, public OSVR_Imager;

### Calibration

Calibration is complicated by the fact that each device seems to have its own special instance.  For the 3rdTech ceiling tracker and other devices, we ended up using custom message types for each device, which is not very satisfying.  It is interesting to think about whether there are things common to calibration across all device types that could be pulled out into a common interface; perhaps "start calibration" and "calibration finished".  Then some eye trackers and some other devices would expose the calibration interface.

### Losing pupils

This is an issue that happens with the optical tracking systems as markers come into and out of view.  We'll have the same thing on the video-based HDK tracking system.  The current solution is to basically stop sending reports when you can't see the thing.  This is an implicit solution.  You can tell (after 3 seconds) that the problem is due to a connection being lost to the device rather than lost pupils, but this is still not very explicit.  I don't have a good solution to this.

### User profiles

I think this belongs in the Json hierarchy, rather than in the devices themselves.

### Other open issues
- Pupil size looks like an analog.
- Pupil aspect ratio looks like an analog.
- Duration of fixation could be determined by watching the fixation start/stop events on the client and doing the math.

## Device class summary
The eye tracker devices provide detailed information about one or both eyes movement. It includes tracking gaze direction using 2D or 3D coordinates, detecting blink events and others (eye fixation, saccades, etc.). Tracker can support binocular and monocular modes. For binocular mode, it will report data based on sensors for both eyes. In monocular mode, only one sensor will provide data. 
The tracker plugin can choose to implement both types at the same time, so that a binocular tracker would also have two monocular trackers.

### Examples
- SMI Eye Tracking product: <http://www.smivision.com/en/gaze-and-eye-tracking-systems/home.html>
- Arrington Research : <http://www.arringtonresearch.com/>
	

### Relation to other classes
**Factoring**: An eye tracker may contain additional features (for example, streaming eye images) that are best factored into other devices classes, such as Imaging. The eye tracker device interface class is only concerned with reporting gaze and blink data. This means that physiognomy data (such as pupil size, pupil aspect ratio, IPD, IOD, eye relief) should be reported in other interface classes: in particular, as analog channels, with meters as the distance units. The JSON device descriptor provides the ability to describe the semantic meaning of these channels.

An eye tracker that provides 3D eye tracking may also want to report a tracker sensor for each eye. While the data described below as a point and a direction does not fully constrain a 6DOF rigid pose, it can be converted into one by choosing -Z as gaze-forward and assuming Y is up (that the eyes are not rotating in their sockets). This is not an opthamologically-valid assumption but it is valid for many other uses of eye tracking.

**Eye tracking from other classes**: Eye tracking algorithms could be created as analysis plugins taking in input from an imaging interface.


## Overview
The Eye tracker interface is summarized in the following diagram:

![Eye tracker interface class](EyeTrackerIntefaceClass.png)

## Messages
An eye tracker may report Gaze Position, Gaze Direction, or both - this should be described in the device descriptor data. Reporting at least one of those messages is required.

### Gaze Position (2D)
#### Data
- Sensor ID
- A 2D vector containing the user's gaze/point of regard, in normalized display coordinates (each component in the range [0, 1], with the display effectively forming that portion of the X-Y plane in the standard OSVR coordinate system).

#### Rationale
An application may wish to draw directly on the point of the screen that the user is looking at. This is one class of applications for eye tracking data, and applications using it are unlikely to also want 3D data, or at least handled in the same method.

### Gaze Direction (3D)
#### Data
- Sensor ID
- A 3D vector (position) containing gaze base point of the user's respective eye in 3D device coordinates.
- A 3D vector (direction vector) containing the normalized gaze direction of the user's respective eye.

#### Rationale
Describes the user's gaze as a ray. This data can be reported in a tracker sensor in addition to this eye-tracker-specific message.

### Blink
#### Data
- Sensor ID
- Event only, no additional data.

#### Rationale
This is an event that can be reported, but is not required.

### Calibrate
#### Data
- None, or perhaps a request ID

#### Rationale
This is effectively a remote procedure call from an application or server to trigger vendor-specific calibration processes. Unlike the other messages, this is sent by a client and received by the plugin.

This message will eventually be subsumed into a device control/configuration interface suitable for multiple different interface classes, along with policy data in the JSON descriptor indicating when calibration must be performed.

### Calibrate Done
#### Data
- None, or request ID from the corresponding calibration request.

#### Rationale
This message indicates that the RPC Calibration call is completed and the device is ready to use.


## Open issues

- There are additional data that can be retrieved from the tracker, that may be useful in applications which include:
	- Duration + location of fixation
	- Loading/saving user profile, if necessary
- When the tracker "loses" pupils (For example, user takes off the HMD), should a special event be generated to let application know of situation?
- Do all eye trackers that report a 3D gaze direction also provide a point as well?

## Other resources
- <http://en.wikipedia.org/wiki/Eye_tracking>
