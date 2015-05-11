##Quick Summary: 
After reading through the APIs for the devices below, I determined the following types of common gestures (listed from most to less common appearance):

- Swipe (left/right)
- Scroll (up/down)
- Single / Double Tap
- Pinch/Spread fingers (zoom in/out )
- Circle (clockwise / counterclockwise rotation)
- Long press
- Rotate

Additionally there are these gestures which are only common to certain device.

- Circle
- Fist
- Double Tap and drag
- Rest

In addition to pre-set gesture, the APIs allow developers to define their own gestures based on certain patterns such as distance between points, motion acceleration/velocity, relative location to other objects, etc. 
Most devices are only limited to hand and finger gesture tracking while Kinect for example tracks entire body skeleton.

There are two types of gestures (purely semantic difference)

- Continous, such as swipe or pinching
- Discrete, such as single tap

Both types of gestures however require a state (begin/end), (although discrete automatically comes with an end state)

Gesture events are often enumerated such as MYO, NOD and Windows Touch(gesture ID part of gesture struct). Others provide a separate classes for each gesture, or callback function for gesture event. 

Depending on the device functionality, gestures are not limited to just one specific hand, so both hands can be used to signal a gesture. Note that common gestures above are defined as a single hand gesture (left or right). 

Open issues:

- How to send/receive customized gestures?


##Leap motion:
Watches for activity for typical gesture. Within a frame it can detect a gesture (event object) and continue it for multiple frames until it stops

It detects the following gestures:

- Circle
- Swipe
- Key Tap (downward tapping motion)
- Screen Tap (forward tapping motion)
Taps are discrete gestures (always come with a stopping state)
Circles and swipes are continuous gestures
Varying configurations are available for each gesture such as circle radius, swipe length, min tap velocity, etc.)

Each gesture is a separate class (subclass from generic Gesture)

##MYO armband:
Provides gestural data in the form of preset gestures(in their terms "poses"). Also provides information about which arm it is and which way oriented.

It reports data when user changes the gesture. Myo automatically knows when gestures stops

It detects the following gestures

- Rest
- Fist
- Wave in
- Wave out
- Fingers spread
- Double tap

Additionally, gestures can be combined with arm motions using Inertial Measurement Unit (IMU) to create
custom gestures.

Each gesture is enumerated

##Nod ring:
Works in 2 modes: Pointer mode and OpenSpatial mode both of which can detect gestures


- HID mode (requires touching and then releasing ring after moving):
	- Move up, down, left, right
    - Select
    - Exit/Escape

- OpenSpatial (framework to express identity, proximity and gestures). Transports motion and gesture data from device to host. Gesture is a combination of button and OpenSpatial data that lets developer configure magnitude of gesture. Allows greater range of gestures than HID mode because of more elaborate set of primitives
	- Clockwise rotation
    - Counterclockwise rotation
    - Scroll up/Down
    - Swipe left/right
    - Swipe up/down

Each gesture is enumerated 
    
##TUIO: 
Open framework that defines common protocol for tangible multitouch surfaces. Allows transmission of data from interactive surfaces such as touch events. Designed to be an abstraction for interactive surfaces Encodes data from tracker app(which reads sensor data) and transmits to client app (which decodes and sends to display)
Allows to create flexible applications by providing Point class (x, y) for a tabletop surface (or spatial points) which you can use to implement your applications as you see fit. 
You can define custom touch events based on tracker data. It sets up server that sends messages to client 

There are no pre-set gesture but it provides the following attributes:

- Position (x, y, z)
- Angle
- Dimension
- Area, Volume
- Velocity vector
- Rotation velocity
- Motion acceleration
- Rotation acceleration

TUIO cursor object provides:

- Motion acceleration and velocity
- Path (vector)
- Accelerating/decelerating state
- X-axis/Y-axis speed
- Stopped/removed state


##Kinect:
Allows heuristic and machine learning approach (gesture builder). It appears to be more versatile than
the rest of the devices in a way of allowing developers to customize gestures.
Provides gesture recognition from each hand, any body sides(doesn't matter which hand does the gesture) or 
other part of the body

It distinguishes between discrete and continuous gestures.

When Kinect senses a gesture, it provides a detection result: Yes, No, Maybe, Unknown

Kinect doesn't provide pre-set gestures but allows to create custom gestures. You can define you gesture with API commands (such as handOverHead gesture is defined as "When any hand Y coordinate is greater than head Y
coordinate") or define gesture motion by recording yourself performing "new gesture".

There are three categories of gestures (Just a semantic distinction of gestures, not code level) :

- Static (hold some gesture like "okay" sign)
- Dynamic (swipe left / right)
- Continuous (no specific pose but movement interacts with application; for ex. "lifting a box")


##Windows Touch:
Windows support for touch displays. Sends a gesture message, specifying if gesture is beginning or ending

It distinguishes between discrete and continuous gestures.

It detects the following gestures:

- Tap/Double Tap (click/double click equivalent)
- Panning with inertia (scrolling)
- Selection/Drag (mouse drag/select)
- Press and tap  (right click)
- Zoom (using two fingers)
- Rotate (move two fingers in opposing directions)
- Two finger tap 
- Long press
- Flicks (up/down/left/right; quick swipes)

Gesture message (struct) contains an ID which corresponds to a type of gesture

##Android: 
Detects various gestures and events when user places finger(s) on the screen.
Gesture event is defined with a finger down (begin) and finishes when finger lifts up off the screen. 

It detects the following gestures:

- Single/Double Tap
- Down motion movement
- Long press
- Scroll
- Swipe or Drag
- Long press drag
- Double tap drag
- Pinch zoom in/out
- Rotate

Gesture detector class provides a listener which has onEvent methods (onDoubleTap, onLongPress, etc.) which return a boolean.