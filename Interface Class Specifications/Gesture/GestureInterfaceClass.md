# Interface Class Description: Gesture

> Status: In Discussions

## Device class summary
Gesture is a form of non-verbal communication in which bodily actions communicate particular messages, which
includes movements of hand, head, and other parts of the body. There is a range of devices that are able to analyze body movements to detect gestures. It is achieved by tracking the body part movement over certain period of time and analyzing its path, velocity/acceleration, rotation, or measuring electrical activity from muscles to detect poses (gestures). This is usually accomplished with cameras, infrared LEDs, eletrical current monitors or EMG sensors that supply information for analysis and detection of gestures.

### Examples
- Leap Motion : <http://leapmotion.com>
- Thalmic Labs MYO armband : <http://halmic.com> 
- Nod Ring : <http://nod.com>
- Microsoft Kinect : <https://microsoft.com/en-us/kinectforwindows/>
- Windows Gestures : <https://msdn.microsoft.com/en-us/library/windows/desktop/dd940543(v=vs.85).aspx>
- Android Gestures : <https://developer.android.com/training/gestures/index.html>
	

### Relation to other classes
**Factoring**: A gesture can be considered as an action that progresses over period of time. During a gesture a number of attributes correspond to it that will make it up such as joint type (hand, head, etc.), orientation, position, acceleration and velocity. Together these attributes logically make up an gesture entity that provides meaningful information about user's body part(s) pose. Gestures can be semantically separated into two types : discrete and continuous. Discrete gesture is a type of gesture that begins and ends at the same time (immediate) such as Single Tap. Continuous gestures occur over a period of time which includes Swiping or Pinching. 

**From other classes**: To determine what type of gesture is occurring it is necessary to collect
positional and motion data for body parts (joints), therefore it is necessary to use Tracker interface, which provides necessary tracking data.  

## Messages
For majority of the cases, it would be sufficient to provide pre-set gestures such as Swipe, Scroll, etc. However a number of devices support custom gestures that will require raw gesture information (Tracker data). 
One device may be tracking multiple body parts (both hands or entire body) therefore each sensor should be reporting its own gesture data, thus a single device can report multiple gestures.


### Gesture metadata
#### Data
- Gesture ID
- State
- Joint ID
- Sensor number

#### Rationale
- There is a pre-set list of gestures that are common to most devices such as Swipe, Scroll, Singe Tap, etc. This list can be expanded as needed. Therefore gestures will be enumerated. Custom gesture will have a reserved ID such as 0 to indicate that a number of special attributes are available (see below).
- Both types of gestures (discrete and continuous) require a state (begin/end), corresponding to 1 and 0 respectively, that will indicate whether or not the gesture has completed, (although discrete automatically comes with an end state).
- The applications/games will need to determine which body part is performing the gesture. Since different devices are tracking different joints and therefore will have different ID for each joint, we will enumerate each body part to include every required joint. Enumerating the joints that will allow the specify from which body part the gesture is occurring.

	#####Gesture ID
Initially we have identified the following list of pre-set gestures : 

	- Swipe (left/right)
	- Scroll (up/down)
	- Single / Double Tap
	- Pinch/Spread fingers (zoom in/out )
	- Circle (clockwise / counterclockwise rotation)
	- Long press
	- Rotate

---
Guillaume comments : 
- Also open/close hand
- Please consider the following article about common gestures
"User-defined gestures for augmented reality" by  	Thammathip Piumsomboon, Adrian Clark,  	Mark Billinghurst,  	Andy Cockburn
This article is a solid basis about gestures classification
---


	#####Joint ID
In order to indicate which body part is performing the gesture, we defined the following list of body parts

	- Left / Right hand
	- Left / Right thumb
	- Left / Right elbow
	- Left / Right shoulder
	- Head
	- Left / Right knee
	- Left / Right hip
	- Left / Right foot
	- Left / Right ankle
	- Spine base
	- Spine mid
	- Spine shoulder 

--- 
Guillaume comments :
	- missing handtips
	- missing hands fingers
	- maybe using h-anim as a norm to represent skeletons, or choose a subset/simplify it
	- should move that enumeration to Skeleton interface specifications
---

### Pose
#### Data
- Position (3D vector)
- Orientation


Guillaume
	also confidence in a joint data, that information is quite common in SDKs delivered with depth sensors

#### Rationale
-For custom gestures positional data allows to compare the path of the gesture to detect new types of gestures

### Motion (Velocity and Acceleration)
#### Data
- Linear velocity (m/s) (3D vector)
- Incremental rotation per delta time (rotational velocity)
- Delta time for incremental rotation (seconds)
- Linear acceleration, exclusive of gravity (m/s^2) (3D vector)
- Increment in incremental rotation per delta time (unit/normalized quaternion)
- Delta time for rotation (seconds)

#### Rationale
-For custom gestures motion data allows to determine how quickly the gesture had occurred to combine motion and position data and detect new types of gestures


## Open issues
- Expanding a list of pre-set gestures

## Overview

The gesture interface is summarized in the following diagram

![Gesture interface](GestureInterface.png)

