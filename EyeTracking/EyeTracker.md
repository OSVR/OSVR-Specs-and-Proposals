# OSVR Device Integration and Factoring Proposal: Eye Tracker Device

> Prepared by Sensics, Inc. on 2-April-2015

Thank you for your interest in connecting with the OSVR ecosystem! This document lays out a proposed strategy for making your device functionality available in OSVR, including how to factor it into the generic device interface classes.


## Configuration/Instantiation
We recommend that your plugin auto-detects the presence of Eye Tracker Device, that is connected to the HMD and will be transmitting data into OSVR. 
The following fully qualified (that is, including the plugin name) device name is suggested `com_vendorName_DeviceName`. 

Tracker can support binocular and monocular modes. For binocular mode, it will report data based on sensors for both eyes. In monocular mode, only one sensor will provide data. 

The tracker plugin can choose to implement both types at the same time, so that a binocular tracker would also have two monocular trackers.

## Interface Classes
The following interface class is recommended for a first pass at the plugin. It is listed in a proposed order of implementation based on, among other factors, API availability and client application usage.

- Eye Tracker interface: A device instance would be able to report the following info:
	- Gaze direction: Gaze data types are available either in 2D (screen coordinates) or 3D (direction vector) coordinates. The normalized gaze position will provide normalized 0 - 1 coordinates. By default, we'll use normalized pixels and quaternions for 2D and 3D respectively.
	- Blink event	: Boolean event that signals whenever user blinks which is an optional feature
	- Calibration	: Allows the plugin to start calibration process and detect when calibration is done. Calibration can initially be vendor-specific.

Initially, there will be support for binocular mode only (verged gaze point), and monocular mode to track gazes of individual eyes would be added later on. 

## Device Descriptor
OSVR devices provide a JSON device descriptor to the core that describes their capabilities, identifies the interfaces and channels provided in a human-friendly and semantic way, and more.

The OSVR distribution includes a few samples of JSON device descriptors as well as a JSON-Schema for the existing interfaces. A preview version of a web app based on the schema is available to help compose them. OSVR-Core includes a tool that reads in a JSON file, parses and compacts it, and writes out a header file with the resulting contents as a string literal, so you can easily and automatically embed a JSON descriptor maintained as a separate file into your compiled plugin. The example plugin build system includes an example of this functionality.

Let us know if you think there is important metadata about your device that is not accounted for in the current JSON schema. For extensibility, the core does not validate submitted JSON against the schema, so it is a starting point, not a limit, but your observations can help improve the system for all.

## Async Device Style
As a general guideline, we'd recommend creating an `Async` device plugin, which allows your code to block until you receive new data, which you then report. In an `Async` plugin, the OSVR core handles the thread creation and synchronization, safely permitting access to shared OSVR resources automatically.

If your update rate is very high and your SDK provides a means for rapidly checking for updates and returning, a `Sync` plugin (whose callback runs in the server's tight main loop) may be appropriate, but we'd suggest at least starting with an `Async` plugin, and changing to a `Sync` device only at a later date and only if needed. (This involves only one line of code change.)

## Platform Support
OSVR is designed from the ground-up to be cross-platform, from multiple desktop platforms to mobile. The PluginKit API you'll use in your plugin is identical across all platforms, both those currently supported and those we anticipate the community supporting. As such, we encourage you to make your plugin as cross-platform as possible. Using the recommended CMake build system as provided for by the example is a good step.

Presently, our largest user community is on Windows, so a functional plugin there is likely of first priority. However, we recommend taking steps (such as use of CMake) that will make it easy to later add support on other platforms.

If your SDK varies substantially between platforms, you may need to use different code on each platform. We'd encourage you to keep as much code, including the plugin name and JSON descriptors, common between all platforms.

## Collaboration
As your device uses existing interface classes, we are confident that your development experience should be smooth. However, to aid us in our ability to support you, if your plugin is developed in a private GitHub repository, we suggest adding a team with read access to that repository, with the following Sensics staff added to that team:

- Ryan Pavlik		- GitHub username `rpavlik`
- Yuval Boger 		- GitHub username `yboger`
- Georgiy Frolov 	- GitHub username `mars979`

## Open Issues
- There are additional data that can be retrieved from the tracker, that may be useful in applications which include:
	- Pupil size
	- Pupil Aspect Ratio
	- Duration of fixation
	- Loading/saving user profile, if necessary
- When the tracker "loses" pupils (For example, user takes off the HMD), should a special event be generated to let application know of situation?