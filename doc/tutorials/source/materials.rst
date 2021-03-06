Materials: How to change the visual appearance of your objects
==============================================================

Introduction
------------

A Material object is what determines an object's visual appearance.
This includes the color of the object at a particular point on the surface
(or within the volume inside the object) but it may also include the
illumination model that's applied to determine the final pixel color.

How a material is properly described is highly dependent on the
renderer that is used for producing the output image. For example, a
RenderMan renderer allows the user to specify little code fragments,
so called *shaders*, that compute the final surface color whereas an
interactive OpenGL visualization has only a small set of parameters
that define the visual appearance.

As cgkit is independent from any particular renderer, it doesn't
enforce a particular way to describe materials. That's why the most
basic material class called ``Material`` is rather limited in its
functionality.  Well, in fact, it does nothing except storing a name
that identifies the material (this is already enough for the OGRE
viewer that uses the material name as a reference to an external OGRE
specific material script). Every other material class that has
additional functionality is a specialized material that is usually
targeted at a particular renderer.  The ``GLMaterial`` class
represents the standard OpenGL illumination model and the set of
parameters that OpenGL provides and is the material of choice if you
want to visualize your scene using the standard viewer tool.  The
``Material3DS`` represents the material as stored in 3DS files (and
the illumination model of 3D Studio), the ``OBJMaterial`` stores the
material definition of the materials that come along with Wavefront
OBJ files. The RIB exporter, which is used when the scene is rendered
by a RenderMan renderer, has a rather flexible framework to create
RenderMan shaders on the fly from material objects. When the
illumination model of a material is known and can be written in the
RenderMan Shading Language it is possible to have the RIB exporter
generate appropriate shaders. This is why the ``GLMaterial``,
``Material3DS`` and ``OBJMaterial`` objects can also be used in
conjunction with RenderMan. The shader generated from the OpenGL
material will simulate the OpenGL lighting model, producing the same
image than OpenGL would have produced (with the difference that
RenderMan will shade every pixel instead of only every vertex). The
3DS shader on the other hand will approximate the lighting model that
3D Studio has used (note that this is 3D Studio, not 3D Studio
**MAX**, so the ``Material3DS`` class only represents a small subset
of what MAX can do). Of course, when using RenderMan you can also use
the special RenderMan material called ``RMMaterial`` and specify your
own shaders.

Assigning materials
-------------------

cgkit allows you to assign an arbitrary number of materials to one
object. During object construction you can pass a single material or a
sequence of materials via the ``material`` argument. Alternatively,
you can use the ``setNumMaterials()`` and ``setMaterial()`` methods of
the ``WorldObject`` base class.

.. code-block:: python

    # Demonstrate how to assign materials:

    mat1 = GLMaterial(diffuse=(1,0,0))
    mat2 = GLMaterial(diffuse=(0,1,0))

    # Assigning one single material
    Box( material = mat1 )

    # Assigning several materials at once
    Box( material = [mat1, mat2] )

    # Assigning materials after object creation
    b = Box()
    b.setMaterial(mat1)

    b = Box()
    b.setNumMaterials(2)
    b.setMaterial(mat1, 0)
    b.setMaterial(mat2, 1)

If an object has one material assigned the entire object will be
rendered using this material. But what if several materials are
assigned? The builtin functionality in cgkit uses the convention that
the material is selected by the primitive variable ``"uniform
int matid"``. When used on a ``TriMesh`` for example, you can assign
each triangle a different material. If the variable ``matid`` is not
present, the first material is used for the entire object and all additional
materials are ignored.

.. code-block:: python

    # Demonstrate how to *use* more than one material per object:

    GLTargetDistantLight(
        pos = (0.3, -0.5, 1)
    )

    TargetCamera(
        pos = (2, -3, 1.5),
        fov = 30
    )

    # Create a box that has two materials assigned. The first material
    # is green, the second is red.
    b = Box(
        material = [ GLMaterial(diffuse=(0,1,0)),
                     GLMaterial(diffuse=(1,0,0)) ]
    )
    # Convert the box to a trimesh object
    convertToTriMesh(b)
    # Now create a new int variable "matid" on the trimesh
    b.geom.newVariable("matid", UNIFORM, INT)
    # Get the corresponding array slot and set the first two indices to 1
    # (all other indices are still 0 by default).
    # This means the first two triangles in the mesh will use the red material.
    matids = b.geom.slot("matid")
    matids[0] = 1
    matids[1] = 1

The above code produces the following image:

.. image:: pics/multimatbox.jpg


The GLMaterial class
--------------------

This section demonstrates the usage of the ``GLMaterial`` class which
represents the parameters and the lighting model of standard OpenGL v1.x
(version 2 features such as shaders are not yet supported). This material
class is intended for use with the standard OpenGL viewer tool. However,
it can also be used for RenderMan output.

The material is composed of four separate colors:

- Ambient color
- Diffuse color
- Specular color
- Emission color

The diffuse color is what you colloquially refer to the *color* of an
object. The color components are the coefficients for the diffuse reflection
part in the lighting model, i.e. light that is impinging on the surface
is scattered equally in all directions which makes the visual appearance
independent on the position of the viewer. So for example, if you specify
a diffuse color of (1,0,0) then only the red component of the impinging
light is reflected and the green and blue components are absorbed. As
a result, the object appears to be red.

The ambient color brightens up an object by a constant factor that is the
product of the ambient color of the material and the ambient color of the
light sources. You can use the ambient color to brigten up areas that are 
not illuminated and would appear in black.

Here is an example script (:download:`ambientdiffuse.py<files/ambientdiffuse.py>`)
that demonstrates the difference between ambient and diffuse color:

.. code-block:: python

    # Ambient color vs diffuse color

    TargetCamera(
        pos = (0,-10,0),
        fov = 30
    )

    GLTargetDistantLight(
        pos = (0,-1,1),
        ambient = (1,1,1),
        diffuse = (1,1,1),
    )

    for i in range(5):
        v = i/4.0
        for j in range(5):
            u = j/4.0
            Sphere(pos = (j-2,0,i-2),
                   radius = 0.3,
                   segmentsu = 128,
                   segmentsv = 64,
                   material = GLMaterial(ambient = u*vec3(0.5,0.5,1),
                                         diffuse = v*vec3(1,0.5,0.5)))

.. image:: pics/ambientdiffuse.jpg

From left to right, the ambient color is changing from black to light
blue. From bottom to top, the diffuse color is changing from black to
red. Note that in the above example the ambient color of the *light
source* was set to white so that the effect of the ambient color of
the material can clearly be seen.

When only the diffuse (and ambient) color is used the objects will always
appear with a dull surface. If you want them to appear more polished and
shiny you can utilise the specular color. When an object has a shiny 
appearance the light that falls on the surface is mainly reflected into
the reflection direction. Such objects have highlights which are actually
the reflected light sources. In addition to the specular color, which 
determines the color of the highlight, you also have to specify how shiny
a surface is, i.e. how large the highlight will be.

.. image:: pics/specular.jpg

From left to right the shininess is 5, 25, 45, 65 and 85. From bottom
to top the specular color goes from the same color than the diffuse color
to white.

Finally, there is the emission color which is almost the same than the
ambient color, it is just that it doesn't need any light. So it is
really just a constant that's added to the final pixel color.

Transparency
~~~~~~~~~~~~

In OpenGL transparency is achieved by a technique called *alpha blending*.
Without alpha blending an object replaces the pixels in the framebuffer if
it is nearer than what has been rendered up to that point. But when
alpha blending is enabled an output pixel is a weighted sum of the
existing pixel in the framebuffer and the newly calculated pixel color.
How exactly these two colors are combined can be specified by the user
via the *blend factors* (see glBlendFunc_).

.. _glBlendFunc: http://pyopengl.sourceforge.net/documentation/manual-3.0/glBlendFunc.xhtml

To achieve a transparency effect you can use the 4th component of a color,
the so called *alpha value*. This value specifies the opacity of an object
when used with the following blend factors: ``(GL_SRC_ALPHA, 
GL_ONE_MINUS_SRC_ALPHA)``. There is a transparency example in the next
section about texture mapping.

**Note:** Alpha blending can currently only be used in certain
situations. To render transparent objects correctly the triangles
that make up the object have to be sorted so that they are drawn
from back to front. The standard viewer currently doesn't do that.
It only guarantees that an object with alpha blending enabled will
be drawn after an object without alpha blending. Basically this means
transparent objects shouldn't overlap on screen and the objects should
be convex (and backface culling has to be enabled).


Texturing
~~~~~~~~~

Instead of using one single color for the entire object (or for every
vertex of an object) you can map an image onto the object. Here is a
simple example (:download:`texdemo1.py <files/texdemo1.py>`, :download:`uvmap.png <files/uvmap.png>`)
where a box is textured using an image file.

.. code-block:: python

    # Texture mapping example

    Box(
        material = GLMaterial(
                      texture = GLTexture(imagename="uvmap.png")
                   )
    )

The following image is used as texture map (the numbers in the corners
are the texture coordinates):

.. image:: pics/uvmap.png

And the final image looks like this:

.. image:: pics/texbox.jpg

The texture is described via a ``GLTexture`` object and is passed to the
material using the ``texture`` parameter. The ``GLTexture`` contains all
the parameter that influence the appearance of the texture. This includes
the image itself (either a filename, a PIL image or the raw RGB data),
the mipmap settings, an image transformation, etc. Additionally, the
object that is being textured has to have *texture coordinates* assigned.
This means, every vertex not only requires a 3D position, but also a 
"position" inside the texture map, i.e. the 2D texture coordinate. A box has
default texture coordinates, that's why the above example did already
work without explicitly assigning texture coordinates. However, it you
have a triangle mesh for example, then you have to set texture coordinates
via the primitive variable called "st" (see the last section in the
tutorial :doc:`custom_vars`). Usually, you'll also
assign texture coordinates in the same modeling package you've used to
create the model in the first place.

You also have the option to modify the placement of the texture without
touching the texture coordinates and that's via the ``transform`` parameter
of the texture. This parameter takes a 4x4 matrix as input which specifies
a transformation that will be applied to the texture coordinates. This way,
you can move, scale or rotate an image on an object. The following example
is a modification of the above example where the image is shrunk by 1/3
(i.e. the texture coordinates are multiplied by 3).

.. code-block:: python

    # Texture mapping example

    Box(
        material = GLMaterial(
                      texture = GLTexture(
                                    imagename = "uvmap.png",
                                    transform = mat4().scaling(vec3(3,3,3))
                                )
                   )
    )

.. image:: pics/texbox2.jpg

You might have noticed that in the above examples the box wasn't illuminated
anymore. That's the default mode of a texture, the decal mode, where the
color value in the image is directly used as output. There are three more
modes. The following example (:download:`texdemo3.py <files/texdemo3.py>`)
shows three of the four modes::

    # Texture mapping example

    TargetCamera(
        pos = (0, -15, 2),
        fov = 30
    )

    GLTargetDistantLight(
        pos = (-0.2, -1, 0.5)
    )

    CCylinder(
        pos = (-3,0,0),
        material = GLMaterial(
                      texture = GLTexture(
                                    imagename = "uvmap.png",
                                    mode = GL_DECAL
                                )
                   )
    )

    CCylinder(
        pos = (0,0,0),
        material = GLMaterial(
                      diffuse = (1,1,1),
                      texture = GLTexture(
                                    imagename = "uvmap.png",
                                    mode = GL_MODULATE
                                )
                   )
    )

    CCylinder(
        pos = (3,0,0),
        material = GLMaterial(
                      diffuse = (1,0,0),
                      texture = GLTexture(
                                    imagename = "uvmap.png",
                                    mode = GL_BLEND,
                                    texenvcolor = (1,1,0)
                                )
                   )
    )

.. image:: pics/texmodes.jpg

The leftmost object uses the default GL_DECAL mode which means neither
the light source nor the base color of the object is taken into account.
The object in the middle uses the GL_MODULATE mode. In this case, the object
is shaded as if the texture wasn't there and the resulting color is
multiplied by the texture color. When used with a white base color you
just get an illuminated version of the textured object.
The rightmost object uses the GL_BLEND mode. Here, the texture color is
used as a weight to blend between the normal (illuminated) object color
without texture and the color passed in via the ``texenvcolor`` parameter.
In the above example the base color of the object is red and the texenvcolor
is yellow. This means wherever the input texture is dark (e.g. the grid 
lines), the red base color will shine through and wherever the texture
image is bright, the yellow texenvcolor will dominate.
The fourth mode would be GL_REPLACE, but with RGB images (without alpha)
the result is the same than GL_DECAL.

The next example (:download:`texdemo4.py <files/texdemo4.py>`, :download:`alphatex.png <files/alphatex.png>`)
uses an image map that
also contains alpha values which are used for masking out certain
regions of the image. This is done with alpha blending which is
activated by specifiying the ``blend_factors`` parameter with the
appropriate factors.

.. code-block:: python

    # Alpha blending example

    from OpenGL.GL import *
    import Image, ImageDraw

    TargetCamera(
        pos = (0,-3,0),
        fov = 30
    )

    Plane(
        # Rotate the plane so that it lies in the XZ plane
        rot = mat3().fromEulerXYZ(radians(90), 0, 0),

        material = GLMaterial(
                      diffuse = (1,1,1,1),
                      blend_factors = (GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA),
                      texture = GLTexture(
                                    imagename = "alphatex.png",
                                    internalformat = GL_RGBA,
                                    mode = GL_REPLACE,
                                    # mirror the image
                                    transform = mat4().scaling(vec3(1, -1, 1))
                                )
                   )
    )

    # Set some spheres as background
    mat = GLMaterial(diffuse=(0.8,0.8,1.0), emission=(0.3,0.3,0.3))
    for i in range(5):
        for j in range(5):
            x = 0.3*(i-2)
            y = 0.3*(j-2)
            Sphere(pos=(x, 2, y), radius=0.2, material=mat, segmentsu=32, segmentsv=16)

.. image:: pics/alphademo.jpg

Finally, you can also apply image maps as *environment maps*. The
difference to regular texture mapping is the way that colors are
looked up from the image map. So far, you had to specify texture
coordinates to map an image onto a surface. But you can also have
OpenGL calculate the texture coordinates automatically by assuming
that the image is mapped onto a sphere with infinite radius that
surrounds the scene. OpenGL will then calculate the reflection direction
and check which color lies in that direction and use this color for
texturing. The resulting image looks as if the object mirrors its
environment (i.e. the content of the image). All you have to do to
activate environment mapping is setting the parameter ``environment_map``
of the ``GLTexture`` object to ``True`` and that's it.

.. code-block:: python

    mat = GLMaterial(
        diffuse = (1,1,1),
        texture = GLTexture(
                    imagename = "environment.jpg",
                    environment_map = True,
                  )
    )

.. image:: pics/envmap.jpg

