> Status: **initial draft**, last updated 10-Apr-2015

Work area for config interface class

The config class interface is to be implemented with every device.  It provides:

##Events
- Device connected
- Device disconnected
- Low battery
- Device error: Errors are already supported via the vrpn_Text interface built into each device.  There is an integer identifier that goes along with each, which can be used to determine what kind of problem there is.  The above events (connect, disconnect, low battery) could be encoded as warning messages.
- Battery level could be an analog between 0 and 1 that could be added to devices that have batteries in them.  This would be in addition to the low-battery warning.

##Services
-- Initialize device (Russ: This seems like something that the server should do at start-up and when it detects a failure.  Not sure when the client would do this, but we could have a "reset" so that if the client noticed strange behavior they could try it.)
-- Get version information (e.g. firmware, serial number)  (Russ: One job of the OSVR device layer is to hide the need to know this from the client code by factoring and then mapping whatever decisions need to be made based on this into the API.  I view having the need to send this information to the client directly as a failure to appropriately factor and map the devices.)

##Open questions
- What events and services are missing?
- What callbacks are required?  (Lets build it into the vrpn_BaseClass and put virtual methods for all devices to implement if they want.)
- How do we handle device-specific configuration?
- Should there be provision for calibration?  (Russ: This seems like a really good place to do calibration, unless there is a whole separate calibration class.  If there is a config class, I think that calibration should be a part of it.  Most devices may ignore the calibration messages.)
- Should there be provision to save/load config?  (Russ: I see this happening at the Json level, and I think it will be important.  This will be like the controller-setup section of games.  I imagine each person having their own config file that describes their body features (IPD, height) and preferences (handedness) that can pulled down from the web and is associated with their account.)
