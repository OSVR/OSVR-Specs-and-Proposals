# Interface Class Description: Eye Tracker

> status: in discussions

## Device class summary
The eye tracker devices provide detailed information about one or both eyes movement. It includes tracking gaze direction using 2D or 3D coordinates, detecting blink events and others (eye fixation, saccades, etc.). Tracker can support binocular and monocular modes. For binocular mode, it will report data based on sensors for both eyes. In monocular mode, only one sensor will provide data. 
The tracker plugin can choose to implement both types at the same time, so that a binocular tracker would also have two monocular trackers.

### Examples
- SMI Eye Tracking product: <http://www.smivision.com/en/gaze-and-eye-tracking-systems/home.html>
- Arrington Research : <http://www.arringtonresearch.com/>
	

### Relation to other classes
**Factoring**: An eye tracker may contain additional features (For example, streaming eye images) that are best factored into other devices classes, such as Imaging. The eye tracker device class is only concerned with reporting gaze direction, detecting blink events, and few additional features(TBD) .

**Eye tracking from other classes**: Head tracking data can be obtained from tracker devices and used in combination with gaze direction.

## Overview
The Eye tracker interface is summarized in the following diagram:

![Eye tracker interface class](EyeTrackerIntefaceClass.png)

## Messages
The gaze position is going to be offered either in 2D or 3D measured in pixel and mm units respectively.

### Gaze
#### Data
##### Gaze Position
- A 2D vector (screen coordinates) containing the current user's gaze direction, in pixels.

##### Gaze Direction 
- A 3D vector (position) containing gaze base point of the user's respective eye in 3D device coordinates.
- A 3D vector (direction vector) containing the normalized gaze direction of the user's respective eye.

#### Rationale
Allows an application to process gaze input in the most convinient format, thus providing flexibility in choice of coordinates for different types of applications. 


### Blink
#### Data
- Boolean event that will signal whether the blink had occurred 

#### Rationale
Certain trackers provide support for detecting blink events, therefore this feature is optional. 

### Calibration
#### Rationale
Allows the plugin to start calibration process and detect when calibration is done. Calibration can initially be vendor-specific.

An application would likely compute a "delta-position" at its convenience, rather than explicitly using the position directly. Devices could report this themselves (particularly in active devices where this can be directly measured), or an analysis plugin could perform the integration to add these messages to 

### Physiognomy 
#### Data
##### IPD
- float containing the distance betweeen the pupil centers in mm

##### IOD
- float containing the distance betweeen the eye-ball centers

##### Eye relief
- float containing the distance betweeen the pupil center and the HMD's lens

##### Pupil size
- float containing the diameter of the pupil in mm
	

#### Rationale
In order to allow for personalized rendering/ display calibration additional information about the users physiognomy are required.

## Open issues

- There are additional data that can be retrieved from the tracker, that may be useful in applications which include:
	- Pupil Aspect Ratio
	- Duration + location of fixation
	- Loading/saving user profile, if necessary
- When the tracker "loses" pupils (For example, user takes off the HMD), should a special event be generated to let application know of situation?

## Other resources
- <http://en.wikipedia.org/wiki/Eye_tracking>
