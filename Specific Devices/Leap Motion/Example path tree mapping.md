## Example path tree mapping

This is an illustrative subset of an expected mapping into the path tree for the most common usages. The full data would of course be available.

- `/me/hands/left` - Receives reports from the tracker sensor corresponding to the palm
- `/me/hands/left/fingers` - this level is just a container
- `/me/hands/left/fingers/index` - This level could also be just a container, but it would likely be more useful if the full pose of the fingertip was routed to it for convenience.
- `/me/hands/left/fingers/index/joints` - This level contains tracker reports (pose: position + orientation, except for tip which would just be position, or perhaps position and the orientation copied from its parent) specifically for the joints as reported.
- `/me/hands/left/fingers/index/joints/metacarpal_base`
- `/me/hands/left/fingers/index/joints/proximal_phalanx_base`
- `/me/hands/left/fingers/index/joints/intermediate_phalanx_base`
- `/me/hands/left/fingers/index/joints/distal_phalanx_base`
- `/me/hands/left/fingers/index/joints/tip`
- `/me/hands/left/fingers/index/bones` - This level would be generated from the skeleton connectivity data and the joint trackers, providing similar data to your `Bone` class.
- `/me/hands/left/fingers/index/bones/metacarpal`
- `/me/hands/left/fingers/index/bones/proximal_phalanx`
- `/me/hands/left/fingers/index/bones/intermediate_phalanx`
- `/me/hands/left/fingers/index/bones/distal_phalanx`

Note that the path tree here (that is, the one rooted at `/me`) somewhat reflects the anatomical "tree" of bones and joints, but is more specifically focused on a user-developer that is not necessarily interested in the "scenegraph of the human body". You can get directly to the hand without going through elbow and wrist, and if the suggestion regarding the named finger level receiving fingertip data is followed, you can get fingertips without going through all the joints to get there.

Making the full tree available with anatomical connectivity (in another portion of the path tree) is certainly reasonable, though it could be done automatically by the core based on your data and descriptors, and is not as high of a priority.