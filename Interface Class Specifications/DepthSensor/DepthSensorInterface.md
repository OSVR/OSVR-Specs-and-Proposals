# Interface Class Specification: DepthSensor

## Device class summary

> Status: first proposal

The purpose of this interface is to provide a generic way to access and use depth sensors like, Kinect, Kinect2, Asus Xtion, Creative Senz3D, and others.

The depth sensor interface allows to :

	- log a string,

	- detect available depth sensor plugins, 

	- check is a required SDK is available to get the plugin functionning properly, 
	
	- check the number of available depth sensors recognized by a depth sensor plugin,

	- let depth sensor plugins declare their own capabilities since depth sensors don't all provide the same functionalities,
	  A capability may have sub-capabilities.
	  Each capability may provides parameters typed for automation.
	  
	  A mental map for the Kinect 1 For Windows is provided in the repository as the file I3DKinectCapabilitiesMentalMap.png

	- create a device, 
	
	- remove a device,
	
	- set the device feed mode, either push or pull, push is the case of a depth sensor plugin calling a callback when a new event poped, pull when the client using OSVR wants to grab data at a specific time like getting an rgb frame,
	
	- get the parameters of a capability,
	
	- set a parameter of a capability, for example asking for the depth stream to use a specific resolution like 640x480,
	
	- launch a device,
	
	- set callbacks in the case of a pull mode, callbacks may get called with a string parameter which is then a json serialization of data in concern, so far we currently use the following callbacks :
		- new rgb frame,
		- new mapped rgb frame,
		- new depth frame,
		- new infrared frame,
		- new body event,
		- new interaction event,
		- new voice recognition event,
		- new pointcloud event,
	
	- set buffers like a depth frame buffer, quite useful when getting images from sensors, buffers are created by the application not the depth sensor plugins,
	
	- enable a capability,
	
	- enable a sub-capability,
	
	- check if a capability is enabled,
	
	- check is a sub-capability is enabled,
	
	- retrieve data in the case of pull mode : informs the string parameter with serialized data of empty string when buffer (typically for images) is updated.


### Examples
- grabbing from a kinect for windows the last rgb frame mapped to depth space, as the application receiving a callback indicating that the image buffer provided by the application got updated with the mapped rgb frame,
- getting a callback indicating that the pointcloud got updated and its data returned as a string parameter of the callback which is the json serialization of that pointcloud,
- getting informed when a skeleton is detected or updated by the depth sensor,
	

### Relation to other classes
**Factoring**: since data like gestures, skeletons is as well specified by OSVR the json-serialized strings should respect the relevant classes as described by other sections of OSVR interface specifications,

**From other instances of the same interface**: analysis plugins should get access and use of data provided by an heterogeneous system of several depth sensors of different types,

**From other classes**: as soon as other classes are informed of a device ID they should as well either ask for data whenever they want or declare their own callback to depth sensors plugins in concern to get as well informed of a new event relative to data they do need,

**To other classes**: data grabbed should get unserialized to other classes of the OSVR interfaces. 

