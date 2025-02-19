# Overview

The UsdUI schema domain contains schemas for working with node graphs. UsdUI
allows the user to describe where the nodes are with `NodeGraphNodeAPI`, add 
descriptive labels and groups to nodes with `SceneGraphPrimAPI`, and to 
visually organize nodes with Backdrops.

A node graph simplifies the creation complicated networks, such as shading 
networks. UsdUI provides the ability to label prims to support these networks.

(usdUI_working_with_node_graphs)=
## Working With Node Graphs

The `SceneGraphPrimAPI` allows each node to contain a descriptive name 
(`ui:displayName`), a descriptive group (`ui:displayGroup`).

The `NodeGraphNodeAPI` defines how each node appears in the node graph. 
This includes its position (`ui:nodegraph:node:pos`), color
(`ui:nodegraph:node:displayColor`), a link to documentation about the node 
(`ui:nodegraph:node:docURI`), the amount of information the node currently
displays (`ui:nodegraph:node:expansionState`), an icon image to express the
node's intent (`ui:nodegraph:node:icon`), its size (`ui:nodegraph:node:size`),
and its relative depth to other nodes in the graph (`ui:nodegraph:node:stackingOrder`).

Using Backdrops allows nodes to be grouped by region, by underlaying the 
containing nodes and adding a description (`ui:description`). Backdrops would
typically have positions and sizes from the `NodeGraphNodeAPI`.

View the image below to see how these details can be presented, in this case using
ShapeFX Loki.

![Example screenshot](usdUINodeGraph.png)

Below is one way to express the above file in usda

```{code-block} usda
def Material "Material"
{
    token outputs:mtlx:surface.connect = </World/Material/PreviewSurface.outputs:out>

    def Shader "PreviewSurface" (
        prepend apiSchemas = ["NodeGraphNodeAPI"]
    )
    {
        uniform token info:id = "ND_UsdPreviewSurface_surfaceshader"
        color3f inputs:diffuseColor.connect = </World/Material/Color.outputs:out>
        token outputs:out
        uniform color3f ui:nodegraph:node:displayColor = (0.7, 0, 0.7)
        uniform token ui:nodegraph:node:expansionState = "open"
        uniform float2 ui:nodegraph:node:pos = (-.85, 1.9)
    }

    def Shader "Color" (
        prepend apiSchemas = ["NodeGraphNodeAPI"]
    )
    {
        uniform token info:id = "ND_constant_color3"
        color3f inputs:value = (1, 0.023, 0.701)
        color3f outputs:out
        uniform color3f ui:nodegraph:node:displayColor = (0, 0.7, 0.7)
        uniform token ui:nodegraph:node:expansionState = "closed"
        uniform float2 ui:nodegraph:node:pos = (-2, 2)
    }

        def Backdrop "Backdrop" (
        prepend apiSchemas = ["NodeGraphNodeAPI"]
    )
    {
        uniform token ui:description = "Do not edit!"
        uniform color3f ui:nodegraph:node:displayColor = (0.8, 0.5, 0.2)
        uniform float2 ui:nodegraph:node:pos = (-0.8, 0.5)
        uniform float2 ui:nodegraph:node:size = (450, 330)
    }
}

```

While not all possible variations are shown above, the following can be observed::
- `ui:nodegraph:node:displayColor` helps distinguish nodes quickly
- `ui:description` in Backdrop can provide context for regional
groupings in a node graph
- `ui:nodegraph:node:expansionState` controls how much information a node displays, 
note the difference in detail between `PreviewSurface` vs `Color`
- `ui:nodegraph:node:size` and `ui:nodegraph:node:pos` determine the placement and relative positioning of nodes in a node graph.
