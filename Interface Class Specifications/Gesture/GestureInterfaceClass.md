> Status: early draft, last updated 10-Apr-2015

This is a work area for gesture interface class specifications.

A gesture interface communicates high-level gestures such as "fist closed", "thumb up", "pinch", "zoom", "swipe" and so forth.

Several type of devices can detect gestures. These include:

- Hand and finger trackers such as Softkinetic
- Thalmic myo
- Nod ring

Gestures could be hand or finger gestures, but could also be head gestures such as "yes" or "no". Two-handed gestures are also possible.

At face value, this is a simple interface as it generates just a gesture event.

The question becomes, how are these gestures defined. are they:

- Enumerated gestures with a placeholder for user-defined or vendor-defined gestures
- Strings containing the names of gestures

## Overview

![Gesture interface](GestureInterface.png)

