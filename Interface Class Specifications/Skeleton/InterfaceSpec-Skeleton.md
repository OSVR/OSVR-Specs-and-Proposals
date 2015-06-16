# Interface Class Specification: Skeleton Tracking

> Status: **in progress**

## Device class summary
This class of devices often provides tracking of joints, limbs, or body parts as a subset of its functionality. The underlying technology may be varied, from vision-based (Leap Motion), to depth-camera based (Kinect/PrimeSense, Kinect 2), to tracker-based (NaturalPoint OptiTrack, ART motion capture and finger tracking), even to datagloves if broadly construed. In some cases, a model is built in to the vendor software to extrapolate an articulated state from the directly-sensed data.

The primary unifying trait is that rather than providing (or only providing) rigid body tracking, they track an articulation, usually of a human user or a region thereof. On a related topic, because connectivity at a minimum and often anatomical association is reported, there is a desire for access to connectivity and exploration of available anatomically-linked reports (in a consistent state) in order to provide a virtual representation of the user in the client software.

Investigation has uncovered a number of existing vendor-specific skeleton representations, as well as several de-facto standards for inferring anatomical association from rigid-body tracker sensor number.

In a sense, the Skeleton's server interface is primarily about relatively static definitions of articulations, association of Tracker interface reports with joints in those descriptions, conventions regarding the coordinate systems involved, and indicating completion of consistent state update. The client interface additionally provides a technique for traversing and understanding tracker reports, as well as derived state inquiries (getting bone length and orientation, based on two joints or sites) that may provide a more natural interface for the developer. As such, this interface class specification varies from earlier examples by focusing more on API than protocol.

Readers are encouraged to consult the references, particularly the H-ANIM specification.

### Examples
- Hand/finger tracking
	- Leap Motion <https://www.leapmotion.com/>
	- Intel RealSense <http://www.intel.com/content/www/us/en/architecture-and-technology/realsense-overview.html> (See also Creative Sens3D <http://us.creative.com/p/web-cameras/creative-senz3d>)
	- ART Fingertracking <http://www.ar-tracking.com/products/interaction/fingertracking/>
	- More indirectly, bend sensor-based datagloves such as
		- 5DT DataGlove Ultra <http://www.5dt.com/?page_id=34>
- Whole-body Skeleton
	- PrioVR <http://priovr.com/>
	- Kinect/PrimeSense, Kinect 2 <http://www.microsoft.com/en-us/kinectforwindows/>
	- SoftKinetic: provides depth cameras and iisu body/hand tracking separately <http://www.softkinetic.com/>
	- Nod skeletal tracking http://developer.nod.com/blog/Nod-Skeletal-Tracking-Unity/
	- NaturalPoint OptiTrack <https://www.naturalpoint.com/optitrack/products/motive/body/>
	- ART Motion Capture products <http://www.ar-tracking.com/products/motion-capture/art-human/>

### Relation to other classes
**Factoring**: The Skeleton class is most closely related with tracking, but provides connectivity and anatomical association beyond just rigid body transforms.

**Skeleton from other classes**: Many body-tracking systems (particularly as one moves closer to industry) are based on tracking of markers, which may be at the joint or positioned somewhere else on the "bone". Additionally, as the PrioVR and Nod examples indicate, it is possible, with the use of a model, to infer skeleton data from less than 6DOF-tracked markers. While the transformation is not necessarily trivial or accurate, one could also interpret bend sensor glove data to infer tracker and skeleton data, which is often done implicitly by software that uses datagloves or by the vendor's SDKs. Vision-based systems demonstrate that imaging data can be turned into tracked skeletons, whether loosely coupled to hardware (like the RealSense SDK and the SoftKinetic software)  or tightly integrated (LeapMotion, some usage of Kinect).

Skeleton capabilities as described here can be added to existing skeleton-like streams (such as from FAAST) by an analysis plugin monitoring the reports and injecting a skeleton-complete message after observing a full data frame.

**Skeleton to other classes**: As the skeleton interface is essentially construed as a layer over one or more tracker devices, analysis that generates skeleton data necessarily generates tracker data. Tracker data with joints reported at specific sensor numbers to match an existing de-facto standard might also be used, for porting pre-OSVR software to work with OSVR-supported skeleton data providers. Skeleton data can be analyzed to extract "gestures," and existing SDKs often include a few built-in gestures. An important distinction for Skeleton data is that more than other interfaces, it is likely to be directly visualized as a humanoid or anatomical subset.

**Skeleton fusion and transformation**: Multiple providers of skeleton data may exist, at different levels of granularity (for instance, a Kinect and an HMD-mounted Leap Motion controller) or with different fields/points of view (multiple Kinects or Leap Motion controllers). Analysis plugins may be created to take in skeletons and merge them, providing a unified skeleton with full data and high detail (different granularity), wider range of motion (different FoV), or more accuracy (different PoV). Analysis plugins may also associate a skeleton that has left the view of a vision-based system with one that has entered, identifying them as being the "same hand", for instance. Conceivably, one could envision combining a vision-based skeleton tracker with another sensor (Nod ring, for instance) to assign persistent identities to a skeleton and associate them with a real person.

## Device description and granularity - plugin side

- **Description of the articulation structure** is relatively static, and thus included in the device descriptor JSON.

Rationale: Humans do not usually gain or lose joints during safe gameplay. The ability of the JSON to be revised at runtime leaves open the possibility of some future (vision-based?) sensor inferring joints and articulations, but this is not a primary use case.

From the point of view of the device plugin,

- One Skeleton interface sensor **per tracked articulation** (body, hand, etc).

Rationale: Substitution of one skeleton for another, in multi-user tracked scenarios

- One Skeleton interface sensor refers to **one or more Tracker interfaces**...

Rationale: Skeletons may be overlaid on existing trackers, but must refer to at least one tracker to provide the data that isn't connectivity or association.

- ...providing **room-space**...

(Or tracker-space - some sort of "global" to be contrasted with parent-relative. )
Rationale: Usability of joint poses by tracker clients as well as skeleton clients for new devices implementing both Tracker and Skeleton at the same time.

- ...**joint**...

Rationale: This provides commonality between skeleton technologies. If marker-based trackers wish to also expose the marker poses, they may, but they are not directly referenced by or required by the skeleton. Similarly, skeleton solutions that adjust reported tracker data to fit a model may choose to report the raw, uncorrected data as additional tracker sensors, but in that case the skeleton should be referring to the adjusted joint tracking sensors

- ... **poses** (usually)

Rationale: Generally, full poses are desired, since humans bones are not solids of revolution. However, for certain "leaf joints" or "sites" that are useful and available (such as fingertip), a position-only may be reported given that there is not a child joint and bone orientation can be inferred from the parent joint.

- with a **standardized coordinate system**.

Rationale: While no single standard coordinate system convention will fit all client applications, consistent input data makes integration easier.

- Following transmission of a skeleton's tracker poses, a **SkeletonComplete message** should be sent for the corresponding skeleton sensor.

## Messages

### SkeletonComplete
#### Data
- Sensor (skeleton) ID

Should be sent (reliable transport) immediately following the completion of sending all the updated joint poses (again, reliable transport) in the skeleton.

#### Rationale
For consumers of skeleton data that are looking for more than a single joint pose, observing skeleton data in a consistent state (all joints updated) is important to avoid undesirable artifacts (bone stretching). This message provides a checkpoint for such a consistent state, provides the opportunity to coalesce tracking data into the derived skeleton data, and provides a "hook" that can be used to trigger user callbacks for skeleton-oriented views of data.

## Coordinate Systems
The base-relative coordinate systems for a humanoid are proposed to be the following:

- Right-handed coordinate system
- Position at the joint/base of the bone
- *y* axis along the bone, pointing in the direction of the bone
- *z* axis coming out the front of the humanoid in a "T-pose"

This overall diagram shows some sample joint coordinate systems following this convention. (As always, RGB correspond to XYZ.)

![](osvr_skeleton2.png)

The following is a closeup of the right hand with select joints shown. (Note that following from the rules, the previous diagram shows that the left hand has the *x* axis pointing down instead of up.)
![](osvr_skeleton1.png)

In 3D-capable browsers, you may be able to view an [interactive 3D viewer of this illustration](diagram/osvr_skeleton.html).

## Device Plugin Interface
A plugin providing skeleton data shall report all of its tracker joints as described above, then issue a SkeletonComplete message for that skeleton sensor.

## Client Application Interface

## Open issues

## Other resources
- Kinect data:
	- FAAST - closed-source VRPN server, uses either OpenNI or Kinect for Windows <http://projects.ict.usc.edu/mxr/faast/>
	- KVR - Open-source VRPN server, uses Kinect for Windows API, uses compatible "sensor" numbering to FAAST <https://github.com/vancegroup/KVR>
- David Nahon (3DS/Dassault) proposal for whole-body skeleton tracking convention: note that CC BY-SA (without clarification as to how far that extends) is claimed <http://fr.slideshare.net/iVEvangelist/v6-unified-skeleton-for-real-time-mocap>
- VRPN skeleton proposal:
	- Full condensed proposal: <http://lists.unc.edu/read/messages?id=6607009>
	- Issue tracker with more elaborated description: <https://git.cs.unc.edu/redmine/vrpn/issues/71>
	- VRPN skeleton/glove discussion on mailing list <http://lists.unc.edu/read/messages?id=6597925>
- ISO standard based on Jack: <http://en.wikipedia.org/wiki/Humanoid_animation>