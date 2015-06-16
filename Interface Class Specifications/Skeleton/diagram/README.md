# OSVR Skeleton Illustration

This directory contains the original and derived (generated) files for the skeleton coordinate system model used for illustration. Some generated files are included in version control here for ease of use/browsing online, and should be updated whenever the input is updated.

If you intend to edit the model, see the [Conversion/Export Process Flowchart](ConversionProcesses.svg).

## Files

### Input
- `osvr_skeleton.blend` - This is the master source file, created with Blender 2.74, based on a ["manikin" model by Blendot from BlendSwap](http://www.blendswap.com/blends/view/931) (under CC0 - see `Original-Model-BLENDSWAP_LICENSE.txt`)

### Output
- In the parent directory, [`osvr_skeleton1.png`](../osvr_skeleton1.png) and [`osvr_skeleton2.png`](../osvr_skeleton2.png) - Output of "Render Animation" from Blender 2.74 (see Model Notes)
- [`osvr_skeleton.html`](osvr_skeleton.html) (**warning: ~3.25 MB file**) - Interactive HTML viewer of the 3D model. Output from converting the X3D export from Blender with `generate-html-from-x3d.cmd`, using InstantReality version 2.5.1.

### Scripts, intermediates, other

- [`Original-Model-BLENDSWAP_LICENSE.txt`](Original-Model-BLENDSWAP_LICENSE.txt) - License/source detail of starting model (humanoid figure).
- `osvr_skeleton.x3d` - X3D export of the model from Blender 2.74 as an intermediate format to generate  web viewer, but may be loaded and viewed in an X3D viewer on its own. 
- `generate-html-from-x3d.cmd` - Windows batch file to convert X3D export to a self-contained HTML file with the model for exploration. Uses the `aopt` tool from [InstantReality](http://www.instantreality.org/downloads/) installed to its default location to perform the conversion, per the [X3DOM 3D data conversion][] instructions.
- `ConversionProcess.graphml` - Flowchart created in [yEd][] showing how to generate the output files from updating the input files.
- [`ConversionProcess.svg`](ConversionProcesses.svg) - SVG export from the `.graphml` file above.

[X3DOM 3D data conversion]: http://www.x3dom.org/documentation/tutorials/generic-3d-data-conversion/
[yEd]: 

## Model Notes

As noted above, a CC0-licensed model was used as a starting point for the Blender file included here. The original was modified (display parameters, materials, etc. changed) and augmented with the coordinate axes for illustration. Other notes:

- The original model had German node names (with diacritics, etc) that caused errors in the exporters - the non-ASCII characters were simply edited out so export could complete.
- The model contains rigging for the humanoid figure, but it is unused and *does not* reflect the OSVR skeleton in any way: the model is for display purposes only.
- The model contains a two-frame "animation" to be able to export stills from the two separate cameras (overall and closeup of hand) easily.