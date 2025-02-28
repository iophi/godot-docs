.. _doc_arraymesh:

Using the ArrayMesh
===================

This tutorial will present the basics of using an :ref:`ArrayMesh <class_arraymesh>`.

To do so, we will use the function :ref:`add_surface_from_arrays() <class_ArrayMesh_method_add_surface_from_arrays>`,
which takes up to five parameters. The first two are required, while the last three are optional.

The first parameter is the ``PrimitiveType``, an OpenGL concept that instructs the GPU
how to arrange the primitive based on the vertices given, i.e. whether they represent triangles,
lines, points, etc. See :ref:`Mesh.PrimitiveType <enum_Mesh_PrimitiveType>` for the options available.

The second parameter, ``arrays``, is the actual Array that stores the mesh information. The array is a normal Godot array that
is constructed with empty brackets ``[]``. It stores a ``Packed**Array`` (e.g. PackedVector3Array,
PackedInt32Array, etc.) for each type of information that will be used to build the surface.

Common elements of ``arrays`` are listed below, together with the position they must have within ``arrays``.
See :ref:`Mesh.ArrayType <enum_Mesh_ArrayType>` for a full list.


.. list-table::
    :class: wrap-normal
    :width: 100%
    :widths: auto
    :header-rows: 1

    * - Index
      - Mesh.ArrayType Enum
      - Array type
    
    * - 0
      - ``ARRAY_VERTEX``
      - :ref:`PackedVector3Array <class_PackedVector3Array>` or :ref:`PackedVector2Array <class_PackedVector2Array>`
    
    * - 1
      - ``ARRAY_NORMAL``
      - :ref:`PackedVector3Array <class_PackedVector3Array>`
    
    * - 2
      - ``ARRAY_TANGENT``
      - :ref:`PackedFloat32Array <class_PackedFloat32Array>` of groups of 4 floats. The first 3 floats determine the tangent, and the last float the binormal 
        direction as -1 or 1.
    
    * - 3
      - ``ARRAY_COLOR``
      - :ref:`PackedColorArray <class_PackedColorArray>`
    
    * - 4
      - ``ARRAY_TEX_UV``
      - :ref:`PackedVector2Array <class_PackedVector2Array>` or :ref:`PackedVector3Array <class_PackedVector3Array>`
    
    * - 5
      - ``ARRAY_TEX_UV2``
      - :ref:`PackedVector2Array <class_PackedVector2Array>` or :ref:`PackedVector3Array <class_PackedVector3Array>`
    
    * - 10
      - ``ARRAY_BONES``
      - :ref:`PackedFloat32Array <class_PackedFloat32Array>` of groups of 4 floats or :ref:`PackedInt32Array <class_PackedInt32Array>` of groups of 4 ints. Each group lists indexes of 4 bones that affects a given vertex.
    
    * - 11
      - ``ARRAY_WEIGHTS``
      - :ref:`PackedFloat32Array <class_PackedFloat32Array>` of groups of 4 floats. Each float lists the amount of weight the corresponding bone in ``ARRAY_BONES`` has on a given vertex.
    
    * - 12
      - ``ARRAY_INDEX``
      - :ref:`PackedInt32Array <class_PackedInt32Array>`

In most cases when creating a mesh, we define it by its vertex positions. So usually, the array of vertices (at index 0) is required, while the index array (at index 12) is optional and
will only be used if included. It is also possible to create a mesh with only the index array and no vertex array, but that's beyond the scope of this tutorial. In fact, we won't use the
index array at all.

All the other arrays carry information about the vertices. They are optional and will only be used if included. Some of these arrays (e.g. ``ARRAY_COLOR``)
use one entry per vertex to provide extra information about vertices. They must have the same size as the vertex array. Other arrays (e.g. ``ARRAY_TANGENT``) use
four entries to describe a single vertex. These must be exactly four times larger than the vertex array.

For normal usage, the last three parameters in :ref:`add_surface_from_arrays() <class_arraymesh_method_add_surface_from_arrays>` are typically left empty.

Setting up the ArrayMesh
------------------------

In the editor, create a :ref:`MeshInstance3D <class_meshinstance3d>` and add an :ref:`ArrayMesh <class_arraymesh>` to it in the Inspector.
Normally, adding an ArrayMesh in the editor is not useful, but in this case it allows us to access the ArrayMesh
from code without creating one.

Next, add a script to the MeshInstance3D.

Under ``_ready()``, create a new Array.

.. tabs::
  .. code-tab:: gdscript GDScript

    var surface_array = []

This will be the array that we keep our surface information in - it will hold
all the arrays of data that the surface needs. Godot will expect it to be of
size ``Mesh.ARRAY_MAX``, so resize it accordingly.

.. tabs::
 .. code-tab:: gdscript GDScript

    var surface_array = []
    surface_array.resize(Mesh.ARRAY_MAX)

Next create the arrays for each data type you will use.

.. tabs::
 .. code-tab:: gdscript GDScript

    var verts = PackedVector3Array()
    var uvs = PackedVector2Array()
    var normals = PackedVector3Array()
    var indices = PackedInt32Array()

Once you have filled your data arrays with your geometry you can create a mesh
by adding each array to ``surface_array`` and then committing to the mesh.

.. tabs::
 .. code-tab:: gdscript GDScript

    surface_array[Mesh.ARRAY_VERTEX] = verts
    surface_array[Mesh.ARRAY_TEX_UV] = uvs
    surface_array[Mesh.ARRAY_NORMAL] = normals
    surface_array[Mesh.ARRAY_INDEX] = indices

    mesh.add_surface_from_arrays(Mesh.PRIMITIVE_TRIANGLES, surface_array) # No blendshapes, lods, or compression used.

.. note:: In this example, we used ``Mesh.PRIMITIVE_TRIANGLES``, but you can use any primitive type
          available from mesh.

Put together, the full code looks like:

.. tabs::
 .. code-tab:: gdscript GDScript

    extends MeshInstance3D

    func _ready():
        var surface_array = []
        surface_array.resize(Mesh.ARRAY_MAX)

        # PackedVector**Arrays for mesh construction.
        var verts = PackedVector3Array()
        var uvs = PackedVector2Array()
        var normals = PackedVector3Array()
        var indices = PackedInt32Array()

        #######################################
        ## Insert code here to generate mesh ##
        #######################################

        # Assign arrays to surface array.
        surface_array[Mesh.ARRAY_VERTEX] = verts
        surface_array[Mesh.ARRAY_TEX_UV] = uvs
        surface_array[Mesh.ARRAY_NORMAL] = normals
        surface_array[Mesh.ARRAY_INDEX] = indices

        # Create mesh surface from mesh array.
        mesh.add_surface_from_arrays(Mesh.PRIMITIVE_TRIANGLES, surface_array) # No blendshapes, lods, or compression used.


The code that goes in the middle can be whatever you want. Below we will present some
example code for generating a sphere.

Generating geometry
-------------------

Here is sample code for generating a sphere. Although the code is presented in
GDScript, there is nothing Godot specific about the approach to generating it.
This implementation has nothing in particular to do with ArrayMeshes and is just a
generic approach to generating a sphere. If you are having trouble understanding it
or want to learn more about procedural geometry in general, you can use any tutorial
that you find online.

.. tabs::
 .. code-tab:: gdscript GDScript

    extends MeshInstance3D

    var rings = 50
    var radial_segments = 50
    var height = 1
    var radius = 1

    func _ready():

        # Insert setting up the PackedVector**Arrays here.

        # Vertex indices.
        var thisrow = 0
        var prevrow = 0
        var point = 0

        # Loop over rings.
        for i in range(rings + 1):
            var v = float(i) / rings
            var w = sin(PI * v)
            var y = cos(PI * v)

            # Loop over segments in ring.
            for j in range(radial_segments):
                var u = float(j) / radial_segments
                var x = sin(u * PI * 2.0)
                var z = cos(u * PI * 2.0)
                var vert = Vector3(x * radius * w, y, z * radius * w)
                verts.append(vert)
                normals.append(vert.normalized())
                uvs.append(Vector2(u, v))
                point += 1

                # Create triangles in ring using indices.
                if i > 0 and j > 0:
                    indices.append(prevrow + j - 1)
                    indices.append(prevrow + j)
                    indices.append(thisrow + j - 1)

                    indices.append(prevrow + j)
                    indices.append(thisrow + j)
                    indices.append(thisrow + j - 1)

            if i > 0:
                indices.append(prevrow + radial_segments - 1)
                indices.append(prevrow)
                indices.append(thisrow + radial_segments - 1)

                indices.append(prevrow)
                indices.append(prevrow + radial_segments)
                indices.append(thisrow + radial_segments - 1)

            prevrow = thisrow
            thisrow = point

      # Insert committing to the ArrayMesh here.

Saving
------

Finally, we can use the :ref:`ResourceSaver <class_resourcesaver>` class to save the ArrayMesh.
This is useful when you want to generate a mesh and then use it later without having to re-generate it.

.. tabs::
 .. code-tab:: gdscript GDScript

    # Saves mesh to a .tres file with compression enabled.
    ResourceSaver.save(mesh, "res://sphere.tres", ResourceSaver.FLAG_COMPRESS)
