- 👋 Hi, I’m @Doggfather254
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Doggfather254/Doggfather254 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import bpy

# Ensure an object is selected
if bpy.context.object is None or bpy.context.object.type != 'MESH':
    print("Please select a mesh object before running this script.")
else:
    obj = bpy.context.object
    
    # Add a Geometry Nodes modifier
    if "3D_Printing_Effect" not in obj.modifiers:
        geo_nodes = obj.modifiers.new(name="3D_Printing_Effect", type='NODES')
    else:
        geo_nodes = obj.modifiers["3D_Printing_Effect"]

    # Create a new node group for the geometry nodes modifier
    if geo_nodes.node_group is None:
        geo_nodes.node_group = bpy.data.node_groups.new(name="3D_Printing_Effect", type='GeometryNodeTree')
        
    node_group = geo_nodes.node_group
    node_group.nodes.clear()  # Clear existing nodes

    # Add input and output nodes
    input_node = node_group.nodes.new('NodeGroupInput')
    output_node = node_group.nodes.new('NodeGroupOutput')
    node_group.inputs.new('NodeSocketGeometry', 'Geometry')
    node_group.outputs.new('NodeSocketGeometry', 'Geometry')

    # Position nodes
    input_node.location = (-500, 0)
    output_node.location = (500, 0)

    # Add nodes for the effect
    mesh_to_points = node_group.nodes.new('GeometryNodeMeshToPoints')
    mesh_to_points.location = (-250, 100)
    mesh_to_points.mode = 'POINTS'

    point_instance = node_group.nodes.new('GeometryNodePointInstance')
    point_instance.location = (100, 100)
    point_instance.instance_type = 'OBJECT'

    cube = bpy.data.objects.get("Cube")
    if not cube:
        # Add a cube to use as the instance object
        bpy.ops.mesh.primitive_cube_add(size=0.1, location=(0, 0, 0))
        cube = bpy.context.object
        cube.name = "Cube"
        cube.hide_set(True)  # Hide the cube object in the viewport

    point_instance.inputs["Instance"].default_value = cube

    # Add Z-position slicing effect
    position_node = node_group.nodes.new('GeometryNodeInputPosition')
    position_node.location = (-250, -100)

    separate_xyz = node_group.nodes.new('ShaderNodeSeparateXYZ')
    separate_xyz.location = (0, -100)

    compare = node_group.nodes.new('FunctionNodeCompare')
    compare.data_type = 'FLOAT'
    compare.operation = 'LESS_THAN'
    compare.location = (250, -100)

    value_node = node_group.nodes.new('ShaderNodeValue')
    value_node.location = (50, -300)
    value_node.label = "Layer Height Controller"
    value_node.outputs[0].default_value = 0.0

    # Connect nodes
    node_group.links.new(input_node.outputs[0], mesh_to_points.inputs[0])
    node_group.links.new(mesh_to_points.outputs[0], point_instance.inputs[0])
    node_group.links.new(point_instance.outputs[0], output_node.inputs[0])

    node_group.links.new(position_node.outputs[0], separate_xyz.inputs[0])
    node_group.links.new(separate_xyz.outputs['Z'], compare.inputs[1])
    node_group.links.new(value_node.outputs[0], compare.inputs[2])
    node_group.links.new(compare.outputs[0], point_instance.inputs[1])

    print("3D printing effect setup completed. Adjust the 'Layer Height Controller' value to animate the effect!")
    
