# Overview

The `UsdSkel` schema domain contains schemas to encode simple skeletons and 
blend shapes. `UsdSkel` is not intended to provide general rigging and 
execution behaviors.


(usdSkel_working_with_joint_animations)=
## Working With Joint based Animations

Working with joint animations 



(usdSkel_working_with_blendshape_animations)=
## Working With Blend shape based Animations

The example below shows an example for how blendshapes can be set up.

```{code-block} usda

#usda 1.0
(
    defaultPrim = "World"
    endTimeCode = 17
    metersPerUnit = 0.01
    startTimeCode = 1
    upAxis = "Y"
)

def Xform "World"
{
    def SkelRoot "MorphingTri" (
        prepend apiSchemas = ["SkelBindingAPI"]
    )
    {
        rel skel:skeleton = </World/MorphingTri/Skel>
        def Skeleton "Skel" {}

        rel skel:animationSource = </World/MorphingTri/Anim>
        def SkelAnimation "Anim"
        {
            uniform token[] blendShapes = ["iso", "right"]
            float[] blendShapeWeights.timeSamples = {
                1: [0, 0],
                5: [1, 0],
                9: [0, 0],
                13: [0, 1],
                17: [1, 1],
            }
        }

        def Mesh "Mesh" (
            prepend apiSchemas = ["SkelBindingAPI"]
        )
        {
            point3f[] points = [(0.5, 0, 0), (0, 0.8660254, 0), (-0.5, 0, 0)]

            uniform token[] skel:blendShapes = ["iso", "right"]
            rel skel:blendShapeTargets = [
                </World/MorphingTri/Mesh/iso>,
                </World/MorphingTri/Mesh/right>,
            ]
            def BlendShape "iso"
            {
                uniform vector3f[] offsets = [(0, 0.8660254, 0)]
                uniform int[] pointIndices = [1]
            }
            def BlendShape "right"
            {
                uniform vector3f[] offsets = [(-0.5, 0.1339746, 0)]
                uniform int[] pointIndices = [1]
            }


            float3[] extent = [(-0.5, 0, 0), (0.5, 0.8660254, 0)]
            int[] faceVertexCounts = [3]
            int[] faceVertexIndices = [0, 1, 2]
        }
    }
}
```

Here is the output of this animation:

![Example screenshot](blendshape.gif)

Here is another version but slowed down and labeled for clarity:

![Example screenshot](blendshape_slowed.gif)

Notice the following from the animation and how the file is structured:
- `Mesh` applies the `SkelBindingAPI` and is parented by a `SkelRoot`. These 
properties describe that the mesh will be considered in a UsdSkel hierarchy.
- `Mesh` parents two defined `BlendShape` prims, `iso` and `right`. These prims 
define their desired shapes in terms of `offsets`. These offsets directly relate 
to the parented mesh `points` array and lengths matter. The length of the offsets 
array must match the length of the pointIndices array. The values in the pointIndices 
array must be a valid index into the points array

(usdSkel_instancing_example)=
## Instancing Example