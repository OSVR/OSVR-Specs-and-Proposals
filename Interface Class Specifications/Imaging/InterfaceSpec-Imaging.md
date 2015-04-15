# Interface Class Specification: Imaging

> Status: **implemented**, though limited to 8 bits-per-channel at this time

## Device class summary
This interface class encompasses a range of devices that report one or more roughly-image-like data streams. They may be (RGB) web cameras, infrared (single channel) cameras, depth maps, etc.

The Imaging interface class builds on OpenCV and its type description and support. As such, it is important to note that **channels and sensors are not synonymous** in Imaging: a color webcam might have 3 8-bit unsigned integer channels, but those 3 *channels* are part of a single image and come from a single *sensor*. An example of a device with multiple sensors would be an integrated unit with both an RGB sensor and a depth/ToF camera.

### Examples
- Webcams supported by OpenCV on any OSVR platform
- Depth camera data
	

### Relation to other classes
**Factoring**: The closest defined class to Imaging would be Analog, but generally the arrangement of data (2D) and quantity of data points make the distinction clear. While depth camera data can be sent with Imaging, it's not designed for general use with point clouds. (Actually, the closest thing to imaging is the vrpn_Imager, and the design discussion below is coming closer to wanting the full generality it was designed to support, incuding cameras with one channel as float (depth) and others as 8-bit color. It would need to be augmented to avoid sending images across the wire if there was no receiver and we'd need to reconsider if we want to open Pandora's box of mixed-mode images (3 color channels encoded in an interleaved fashion). See https://github.com/vrpn/vrpn.github.io/blob/source/source/assets/vrpn_Imager_talk_june_2008.ppt for more info.)

**From other instances of the same interface**: Filters could be applied to imaging data, though setting up an extended data flow pipeline of filters individually in OSVR is not a use case optimized for.

**To other classes**: Imaging can be a source of tracker and skeleton data, in conjunction with software and an internal model.

## Messages
Note that an image message is a logical unit and will be discussed as such, although it may be split and arrive in packets if forced to cross a network interface.

### Imaging Data
#### Data
- Sensor ID
- Metadata:
	- Height in pixels
	- Width in pixels
	- Channels per pixel
	- Byte depth of each channel (1, 2, or 4)
	- Type enum choosing signed int, unsigned int, or floating point
	- Byte order identifier
- Data buffer

#### Rationale
With the exception of the Sensor ID (specified by plugin) and byte order identifier (internally computed), all the data is extracted out of an OpenCV `cv::Mat` structure automatically in the C++ wrapper of PluginKit. Similarly, on the client side, the C++ ClientKit wrapper handles raw callbacks and generates image buffers and `cv::Mat` headers.

In an implementation detail, the "wire" format for these messages is the original host's byte order, hence the inclusion of a byte order identifier. The client library is responsible for changing the byte order if necessary. This avoid unnecessary work in the most common, broadly-homogenous use environments.

## Open issues

- Finishing implementation of multi-byte channels
- Shared memory local interprocess transport
