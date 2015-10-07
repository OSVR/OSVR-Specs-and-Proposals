# Interface Class Description: Gesture

> Status: Completed

## Device class summary
Gesture is a form of non-verbal communication in which bodily actions communicate particular messages, which
includes movements of hand, head, and other parts of the body. There is a range of devices that are able to analyze body movements to detect gestures. It is achieved by tracking the body part movement over certain period of time and analyzing its path, velocity/acceleration, rotation, or measuring electrical activity from muscles to detect poses (gestures). This is usually accomplished with cameras, infrared LEDs, eletrical current monitors or EMG sensors that supply information for analysis and detection of gestures.

### Examples
- Leap Motion : <http://leapmotion.com>
- Thalmic Labs MYO armband : <http://thalmic.com> 
- Nod Ring : <http://nod.com>
- Microsoft Kinect : <https://microsoft.com/en-us/kinectforwindows/>
- Windows Gestures : <https://msdn.microsoft.com/en-us/library/windows/desktop/dd940543(v=vs.85).aspx>
- Android Gestures : <https://developer.android.com/training/gestures/index.html>
	

### Relation to other classes
**Factoring**: A gesture can be considered as an action that progresses over period of time. During a gesture a number of attributes correspond to it that will make it up such as joint type (hand, head, etc.), orientation, position, acceleration and velocity. Together these attributes logically make up a gesture entity that provides meaningful information about user's body part(s) pose. Gestures can be semantically separated into two types : discrete and continuous. Discrete gesture is a type of gesture that begins and ends at the same time (immediate) such as Single Tap. Continuous gestures occur over a period of time which includes Swiping or Pinching.  

## Messages
For majority of the cases, it would be sufficient to provide pre-set gestures such as Swipe, Scroll, etc. However a number of devices support custom gestures. One device may be tracking multiple body parts (both hands or entire body) therefore each sensor should be reporting its own gesture data, thus a single device can report multiple gestures.


### Gesture Metadata
#### Data
- Gesture ID, either an ID from pre-set gestures or a custom ID generated when user defines their own message
- State, either completed or in progress indicating that a gesture is continuous such as Circle
- Sensor ID

#### Rationale
- There is a pre-set list of gestures that are common to most devices such as Swipe, Scroll, Singe Tap, etc. This list can be expanded as needed. Therefore gestures will be integer identified strings. This will allow developers to use pre-set gestures as well as easily expand gestures that haven't been added to pre-set list.
- Both types of gestures (discrete and continuous) require a state (begin/end), corresponding to 1 and 0 respectively, that will indicate whether or not the gesture has completed, (although discrete automatically comes with an end state).
- The applications/games will need to determine which body part is performing the gesture. Since different devices are tracking different joints, a sensor ID can be mapped to a specific joint (in case of multiple tracked joints) and use semantic name to map sensor number to joint name.

####Gesture Names
Initially we have identified the following list of pre-set gestures : 

	- Swipe Left
	- Swipe Right
	- Scroll Up
	- Scroll Down
	- Single Tap
	- Double Tap
	- Pinch
	- Finger Spread
	- Circle
	- Long Press
	- Open Hand
	- Closed Hand

###Custom Gesture Names
In addition to pre-set gesture names above, a device plugin can define its own set of gestures. For each new gesture name, system will generate a new Gesture ID. Client application can retrieve gesture names using the Gesture ID from the report.

## Open issues


## Overview

The gesture interface is summarized in the following diagram

![Gesture interface](GestureInterface.png)

