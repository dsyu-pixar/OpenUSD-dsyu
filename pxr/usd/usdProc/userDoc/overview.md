# Overview

UsdProc allows users to effectively handle cases when a prim is more easily defined by a procedure rather than a static asset. In simpler terms, this API allows a user to send descriptive inputs into another system to generate a prim at runtime.

(usdProc_working_with_proc)=
## Working With Proc

Common use-cases for utilizing UsdProc are:
- To create many scattered objects based on a description, instead of defining all objects individually
    - For example: Trees, Rocks, etc.
- To create structured geometry from a list of params
    - For example: Stairs, Roads, Buildings, etc.

Some tradeoffs should be considered when using `UsdProc` or procedurals in general. Procedurals offer flexibility to generate the geometry at the last minute, incorporating up-to-date attributes from the rest of the scene, but come at the cost of increasing the scene load time.

(usdProc_example)=
### Example

```
#usda 1.0
(
    defaultPrim = "World"
)

def Xform "World"
{
    def Mesh "Terrain"
    {
        uniform token subdivisionScheme = "none"

        point3f[] points = [
            (-5, 0, -5),
            ( 5, 0, -5),
            ( 5, 0,  5),
            (-5, 0,  5)
        ]

        int[] faceVertexCounts = [4]
        int[] faceVertexIndices = [0, 1, 2, 3]
    }

    def Xform "Prototypes"
    {
        def Mesh "Rock"
        {
            uniform token subdivisionScheme = "none"

            point3f[] points = [
                (-0.3, 0.0, -0.3),
                ( 0.3, 0.0, -0.3),
                ( 0.3, 0.0,  0.3),
                (-0.3, 0.0,  0.3),
                ( 0.0, 0.5,  0.0)
            ]

            int[] faceVertexCounts = [3, 3, 3, 3]
            int[] faceVertexIndices = [
                0, 1, 4,
                1, 2, 4,
                2, 3, 4,
                3, 0, 4
            ]
        }
    }

    def "RockScatter" (
        prepend apiSchemas = ["UsdHydraGenerativeProceduralAPI"]
    )
    {
        token proceduralSystem = "hydraGenerativeProcedural"
        uniform token primvars:hdGp:proceduralType = "RockScatter"

        int primvars:rockCount = 200
        double primvars:seed = 42.0
        asset primvars:sourceMesh = </World/Prototypes/Rock>
        asset primvars:terrain = </World/Terrain>
        float primvars:radius = 10.0
    }
}
```

Consider the above example.The focus is on `</World/RockScatter>`. 

It can be seen that this prim is *describing* how to scatter rocks across `</World/Terrain>` utilizing the features of `UsdProc`. 

`UsdProc` requires a `proceduralSystem` to determine which system is responsible for generating the resulting Rock prims. Once defined, a `proceduralType` is defined so that `proceduralSystem` knows what to invoke for this prim. Finally, the remaining primvars of `rockCount`, `seed`, `sourceMesh`, `terrain`, `radius` serve as inputs that the `proceduralSystem` uses to generate the resulting prims as desired.

UsdProc itself is purely a declarative schema and does not define how procedural prims are executed. Hydra provides a plugin-based mechanism for resolving UsdProc prims but the schema is not limited to Hydra. Other resolvers may also interpret and resolve UsdProc prims according to the declared `proceduralSystem`.
