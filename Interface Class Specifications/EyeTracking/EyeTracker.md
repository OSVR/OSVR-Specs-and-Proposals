# Interface Class Description: Eye Tracker

## Device class summary
The eye tracker devices provide detailed information about one or both eyes movement. It includes tracking gaze direction using 2D or 3D coordinates, detecting blink events and others (eye fixation, saccades, etc.). Tracker can support binocular and monocular modes. For binocular mode, it will report data based on sensors for both eyes. In monocular mode, only one sensor will provide data. 
The tracker plugin can choose to implement both types at the same time, so that a binocular tracker would also have two monocular trackers.

### Examples
- SMI Eye Tracking product: <http://www.smivision.com/en/gaze-and-eye-tracking-systems/home.html>
- Arrington Research : <http://www.arringtonresearch.com/>
	

### Relation to other classes
**Factoring**: An eye tracker may contain additional features (For example, streaming eye images) that are best factored into other devices classes, such as Imaging. The eye tracker device class is only concerned with reporting gaze direction, detecting blink events, and few additional features(TBD) .

**Eye tracking from other classes**: Head tracking data can be obtained from tracker devices and used in combination with gaze direction.

## Messages
The gaze position is going to be offered either in 2D or 3D measured in pixel and mm units respectively.

### Gaze Direction
#### Data
- A 2D vector (screen coordinates) containing the current user's gaze direction, in pixels.
- A 3D vector (direction vector) containing the current user's gaze direction, in quaternions.

#### Rationale
This allows application how to process gaze input, thus providing flexibility in choice of coordinates for different types of applications. The normalized gaze position will provide normalized 0 - 1 coordinates. By default, we'll use normalized pixels and quaternions for 2D and 3D respectively.

### Blink
#### Data
- Boolean event that will signal whether the blink had occurred 

#### Rationale
Certain trackers provide support for detecting blink events, therefore this feature is optional. Calibration can initially be vendor-specific.

### Calibration
#### Rationale
Allows the plugin to start calibration process and detect when calibration is done.

An application would likely compute a "delta-position" at its convenience, rather than explicitly using the position directly. Devices could report this themselves (particularly in active devices where this can be directly measured), or an analysis plugin could perform the integration to add these messages to 

## Open issues

- There are additional data that can be retrieved from the tracker, that may be useful in applications which include:
	- Pupil size
	- Pupil Aspect Ratio
	- Duration of fixation
	- Loading/saving user profile, if necessary
- When the tracker "loses" pupils (For example, user takes off the HMD), should a special event be generated to let application know of situation?

## Other resources
- <http://en.wikipedia.org/wiki/Eye_tracking>