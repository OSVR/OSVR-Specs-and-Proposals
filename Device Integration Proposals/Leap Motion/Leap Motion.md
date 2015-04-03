# OSVR Device Integration and Factoring Proposal: Leap Motion

> Prepared by Sensics, Inc. on 12-February-2015

Thank you for your involvement in the OSVR ecosystem! This document lays out a proposed strategy for making your device functionality available in OSVR, including how to factor it into the generic device interface classes.


## Configuration/Instantiation
We recommend that your plugin auto-detect the presence of the Leap Motion sensor, instantiating an object responsible for transmitting data into OSVR. Due to the fact that a single controller/sensor can track a changing number of complex objects (hands and arms), we suggest that this single object take the step of registering more than one "device name" in OSVR. The following fully-qualified (that is, including the plugin name) device names are suggested:

- `/com_leapmotion_leapmotioncontroller/LeapControllerX` - This device would be the source of data that is "per-controller", such as imaging/distortion data. From the documentation, it appears that a single controller per system is currently supported but plans exist for multiple. Thus, the `X` is a placeholder for an ID that could either be:
	- a number, sequentially assigned each time the server is loaded, starting at 0
	- based on a controller-specific identifier (such as serial number or a name assigned in your software).
- `/com_leapmotion_leapmotioncontroller/LeapArmX_Y` - Each named device of this format would correspond to a single tracked arm/hand. `X` would be the same identifier used in the controller, while `Y` would be some other distinguishing identifier. (This two-placeholder name assumes that the tracked data is not fused between controllers before it reaches OSVR. Of course, if this assumption does not hold, the naming convention can be changed.)
	- We'd suggest returning to consider how to report "tool tracking" in a later iteration of the plugin.

At least initially, we'd suggest just directly creating two `LeapArm` devices, one for a left arm, if seen, and one for a right arm, if seen. This would permit fairly fixed mapping from device data to the semantic path tree. Long term, we can consider how best to handle arms leaving and entering, and the associated configuration and description challenges that poses.

## Interface Classes
The following interface classes are recommended for a first pass at the plugin. They are listed in a proposed order of implementation based on, among other factors, API availability and client application usage.

- Tracker interface: A LeapArm would report the (world-relative or at least sensor-relative) pose (position and orientation) of each tracked joint. Each joint would be considered a single sensor or channel, and a fixed mapping of channel number to joint should be used and documented. 
	- Since pose derivatives (velocity and acceleration) are not directly measured, those would not be reported, though a consideration for the future might be exposing estimates of these if your internal model makes them available.
- Imaging interface: A LeapController would expose (at least) three sensors of imaging data: two would be single-channel, 8-bit unsigned integer frames corresponding to the camera images (dynamically sized), while one would be a 64x64 (or dynamically based on your API) two-channel 32-bit float frame containing the calibration data. A method will be made available to find out if any analysis plugins or client applications have opened a given interface, so that imaging data can be sent on demand.
- Skeleton interface: A LeapArm would use this interface, which augments the Tracker interface, to expose the bone length and connectedness/hierarchy of the skeleton data computed in the hand/arm model.

At a later date, the following interface classes will become available for enhanced functionality:

- Gesture interface: for exposing gestures recognized
- Device status interface or similar: for reporting "acquired/lost tracking" on LeapArm devices.

## Device Descriptor
OSVR devices provide a JSON device descriptor to the core that describes their capabilities, identifies the interfaces and channels provided in a human-friendly and semantic way, and more. The data from the Leap controller both makes this descriptor important, and makes producing it more challenging than for simpler devices.

The per-controller device descriptor is fairly simple as described above, as it would simply identify the imaging channels. We're open to suggestions as to how the distortion in the image might be described in a generic way in the JSON descriptor, such that consuming plugins and applications could use it easily.

The work of describing the tracking data fully is split between the JSON descriptor, for more static data, and the Skeleton interface. At a minimum it would contain a basic semantic mapping of the joint tracking to the corresponding anatomical feature, for configuration and use of the tracking data in applications not specifically written to use skeletal tracking.

The OSVR distribution includes a few samples of JSON device descriptors as well as a JSON-Schema for the existing interfaces. During the work to produce the Skeleton interface, the schema will be updated to account for the new data; however, do note that the server does not validate your descriptor against the schema, and thus it is a starting point, not a limit, and intentionally somewhat open-ended. A preview version of a web-based editor generated from the schema is available through the OSVR Preview Access site to help compose them. OSVR-Core includes a tool that reads in a JSON file, parses and compacts it, and writes out a header file with the resulting contents as a string literal, so you can easily and automatically embed a JSON descriptor maintained as a separate file into your compiled plugin. The example plugin build system includes an example of this functionality. If runtime generation of descriptors ends up being necessary, the static portion of the data could be loaded this way and simply augmented at runtime using a JSON library. If you do not already have a preferred library for JSON, we recommend JSONCPP, the library used internally by OSVR.

## Async Device Style
As a general guideline, we'd recommend creating an `Async` device plugin, which allows your code to block until you receive new data, which you then report. In an `Async` plugin, the OSVR core handles the thread creation and synchronization, safely permitting access to shared OSVR resources automatically.

As each time the controller returns data, it results in multiple reports, it will likely be advantageous to explicitly take the "sending lock" once reports are prepared to be sent, instead of allowing the framework to acquire and release them with each report. Exposing this ability to plugins is planned functionality, but the device should still be functional without this, so it does not block the start of implementation.

## Platform Support
OSVR is designed from the ground-up to be cross-platform, from multiple desktop platforms to mobile. The PluginKit API you'll use in your plugin is identical across all platforms, both those currently supported and those we anticipate the community supporting. As such, we encourage you to make your plugin as cross-platform as possible. Using the recommended CMake build system as provided for by the example is a good step.

Presently, our largest user community is on Windows, so a functional plugin there is likely of first priority. However, we recommend taking steps (such as use of CMake) that will make it easy to later add support on other platforms.

## Collaboration
Implementation of the initial auto-configuration and tracker reporting should be smooth and can begin immediately. As your device's skeleton interface presents new territory for the OSVR technology, we anticipate mutual benefit from being able to collaborate over code and request review.

On your end, please let us know who on your team will be working on the plugin, so we can ensure they have source-level access to the OSVR-Core libraries. We'd also recommend that you add read access to your repository for our team so we can assist as needed. If your plugin is developed in a private GitHub repository, we suggest adding a team with read access to that repository, with the following Sensics staff added to that team:

- Yuval Boger - GitHub username `yboger`
- Georgiy Frolov - GitHub username `mars979`
- Ryan Pavlik - GitHub username `rpavlik`

## Open Issues
- HMD mode
	- Can/should it be manually switched on?
	- Does it perform the appropriate transformation to maintain the same "global" coordinate system?
- Reporting confidence/uncertainty in the tracked data
- Does it make sense to provide the distortion correction shader in JSON?
- Handling entry/exit from field of view beyond stopping reporting
	- Related: handling the identity of a tracked arm being permanently lost after it moves out of view
- Handling more than two hands
- Tool tracking
