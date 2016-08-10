# Interface Class Specification: Display
> Status: Draft Proposal

## Device class summary
The display interface is responsible for filling the roll previously filled by static display config json in osvr_server_config.json. Where previously the user would be responsible for selecting the correct display configuration, the display interface allows hardware plugins to specify these parameters automatically at runtime.

### Examples
- None


### Relation to other classes
**Factoring**: The display interface represents a collection of individual parameters for a given display, including parameters that don't fit neatly into an existing interface. While it might be possible (and in some cases appropriate) to expose certain display parameters individually as an analog or button type interface, other types of parameters, such as the distortion mesh or the logical display topology (viewer/eye/surface mapping to physical displays and viewports), cannot.

**From other instances of the same interface**: It isn't likely that analysis plugins will consume a display interface and filter or transform any of its parameters, although it remains possible.

**From other classes**: Although not expected to be common, it may be possible to derive a display interface out of a set of other interfaces representing individual parameters.

**To other classes**: If some display parameters are dynamic, and the hardware in question can read the value of the parameter at runtime as it changes, then it may be useful to expose that parameter separately as its own interface. For example, a display's center of projection might be exposed as an `OSVR_Vec2` interface.

## Messages
In general, to work smoothly with the rest of the OSVR system, distance units are meters and velocity units are meters/second.

### Display
#### Data
- type (enum in fixed|hmd)
- vendor (string)
- model: (string)
- product URI (uri-string RFC 3986)
- driver URI (uri-string RFC 3986)
- notes: string
- number of DisplayInputs (whole number)
- number of logical DisplaySurfaces (whole number)
- k1[3] rgb radial distortion supported (boolean, hmd only?) and priority (whole number)
- RGB radial distortion supported (boolean, hmd only?, supercedes k1[3] in most cases) and priority (whole number)
- mono point mesh distortion supported (boolean, hmd only?) and priority (whole number)
- RGB mesh distortion supported (boolean, hmd only?) and priority (whole number)

#### Rationale
The Display message is the root object of the Display interface. It references both the physical DisplayInputs and logical DisplaySurfaces, which define the mapping between them.

### DisplayInput
> one per number of display inputs in Display message.

#### Data
- resolution (width, height in whole numbers)
- vendor PNPID (string)
- product id (string)
- platform-specific hardware instance path to uniquely identify it (??? string?)
- if part of the desktop, origin on the desktop (X and Y in whole numbers)
- On Windows, dx adapter number (??? string?)
- logical id or platform-agnostic identifier (string? or int? referenced by logical topology)
- location and orientation (physical orientation and location? relative to what?)
- eye relief/lense-to-screen distance (if not represented with location/orientation above)
- Scan-out direction (unit OSVR_Vec2)
- Scan-out framerate (analog/float)
- estimated latency to start of render in ms (analog/float)

#### Rationale
The DisplayInput message represents a physical display input.

### DisplaySurface
> one per number of logical display surfaces in Display message

#### Data
- logical id of display input for this surface (from DisplayInput message)
- eye index (whole number)
- viewer index (whole number)
- viewport (relative to this surface's corresponding DisplayInput)
- rotation (enum in landscape|portrait|landscape-flipped|portrait-flipped in clockwise order, relative to corresponding DisplayInput)
- lens center of projection (OSVR_Vec2, HMD only?)
- hidden area mesh (vector of OSVR_Vec2, HMD only?, optional)
- RGB radial distortion center of projection (OSVR_Vec2, HMD only? optional)
- mono point distortion mesh (HMD only?, optional)
- RGB distortion mesh (HMD only?, optional)

#### Rationale
This is one serialization of the logical viewer-eye-surface mapping to physical display input hardware. It assumes one DisplayInput per DisplaySurface, and one combination of (viewer, eye) per surface. If this is not the case, we'll have to serialize these differently.

## Open issues
- Some of the parameters on DisplaySurface are described elsewhere as per-eye but are here listed as per-surface. This is only NOT a problem if there is only one surface per eye, but can we make that assumption?
- Do we need separate message types for "vector of thing"? For instance, do the distortion meshes (mono point and RGB) and hidden area meshes need to be contained in their own message type, or can we serialize them together?
- This seems brittle. If other distortion correction methods are added later, that would be a breaking change between the server and client. How can we make this more future proof?

## Other resources
- https://github.com/OSVR/OSVR-Docs/blob/master/Configuring/projectionAndViewMatrices.md
- https://github.com/OSVR/OSVR-Docs/blob/master/Configuring/distortion.md
