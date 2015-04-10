> Status: **initial draft**, last updated 10-Apr-2015

Work area for config interface class

The config class interface is to be implemented with every device. It provides:

##Events
- Device connected
- Device disconnected
- Low battery
- Device error

##Services
-- Initialize device
-- Get version information (e.g. firmware, serial number)

##Open questions
- What events and services are missing?
- What callbacks are required?
- How do we handle device-specific configuration?
- Should there be provision for calibration?
- Should there be provision to save/load config?
