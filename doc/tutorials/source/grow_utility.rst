Using the grow utility
======================

This tutorial provides a quick introduction into the usage of the
``grow.py`` utility which you can find in the ``utilities`` directory
of the source archive. This tool generates uniformly distributed
point locations on the surface of one or more objects and calls a user
defined function for each such point.

We will start by placing spheres onto the surface of a torus. First, we
need the 3D model that is used as input for generating the points. This
could be in any file format that cgkit can read. For simplicity, we just
create a Python file ``model1.py`` that contains a torus::

    ######################################################################
    # Demonstrating the grow utility
    #
    # File: model1.py
    ######################################################################
    
    # The generated points will lie on the surface of this torus
    Torus(
        major = 1.0,
        minor = 0.3,
        segmentsu = 32,
        segmentsv = 16,
        visible = False
    )
    
    # The following objects are only necessary for rendering:
    
    TargetCamera(
        pos = (0,-3,2),
        target = (0,0,-0.2),
        fov = 40
    )
    
    SpotLight3DS(
        pos = (-2,-3,2),
        target = (0,0,0),
        falloff = 40,
        hotspot = 35,
        shadowed = True,
        shadow_size = 1024,
        shadow_filter = 2.0,
        shadow_bias = 0.001
    )
    
    RIBArchive(
        filename = "spheres.rib",
        material = RMMaterial( surface = RMShader("plastic"), color=(0.5,1,0.5) )
    )

The torus definition at the beginning would be enough for the grow
utility, but we will also use the same file for rendering the result
so we already add a camera, a light source and we also include the
RIB archive that we will generate later on.

Next, we need a function that processes the generated input points.
This function is the equivalent of a shader during rendering. The 
grow utility itself just generates surface points and some additional
data and passes this data to a user defined function where the actual
RIB generation (or whatever) takes place. So create a file ``proc.py``
with the following content::

    # Generating spheres
    #
    # File: proc.py
    
    def proc(P, **keyargs):
        r = 0.02
        RiTransformBegin()
        RiTranslate(P)
        RiSphere(r,-r,r,360)
        RiTransformEnd()

The name of the function must be identical to the file name. This function
simply creates a sphere of radius 0.02 at the generated position.

Finally, we have to tell the grow utility what exactly to do. There
are command line options to do so (see ``grow.py --help``) but we can
save some typing if we stick the options into a file. The parameters
are passed via the cgkit Globals() section, so create a file ``cfg.py``
that looks like this (have a look at the top of the file ``grow.py``
to see a full list of options together with their description)::

    # Config file for the grow utility
    #
    # File: cfg.py
    
    Globals(
        proc = "proc",
        numpoints = 10000,
        RIBname = "spheres.rib"
    )

We could have put that data into the file ``model1.py`` that we
already have, but by separating the configuration we can reuse the
file for other models. Now everything is ready to run the grow utility::

  > grow.py model1.py cfg.py
  Loading "model1.py"...
  Loading "cfg.py"...
  Loading procedure file "proc.py"...
  Input pattern: "*"
  Output RIB file: "spheres.rib"
  #Surface points: 10000
  Preprocessing object "Torus"...
  Converting "st"/"stfaces" into "varying st"...
  Surface area: 11.7204074185
  Total area: 11.7204074185
  Generating 10000 points on "Torus"...
  10000...
  Generation time: 2s

The utility doesn't distinguish between config file and data file, it
just reads anything that cgkit supports and merges the data into the
current scene. 

The result of running the utility is the file ``spheres.rib``. Now
you can render the result using the render tool::

  > render.py model1.py

.. image:: pics/torus1.jpg

Now let's have a closer look on the procedure we were using to place
the spheres::

    def proc(P, **keyargs):
        r = 0.02
        RiTransformBegin()
        RiTranslate(P)
        RiSphere(r,-r,r,360)
        RiTransformEnd()

The name of the procedure must be specified in the Globals() section using
the ``proc`` argument. The file where the grow utility is looking
for the function must have the same name than the function plus
the ``.py`` suffix. The function will be called with a number of
keyword arguments. In the above example, we were only using the
``P`` argument which is the generated surface position as a ``vec3`` 
object. We also could have used other information such as:

- ``P``: Surface position (vec3)
- ``N``: Surface normal (vec3)
- ``F``: Surface frame (mat4)
- ``s,t``: Current texture coordinate (if available)
  
What exactly the procedure does is entirely up to the procedure.
Here, we were creating spheres at the position of the surface point.
It's not uncommon though that you want to orient the object along
the normal, so that it is properly placed onto the surface of the
original object. This can be done either with the normal or just
with the ``F`` argument which contains the full transformation.
For example, the following procedure will place disks that are 
properly oriented so that their normal is identical to the surface
normal. The position of the disk is shifted along the normal by
a "random" amount (using the snoise() function)::

    # Generating disks
    #
    # File: proc2.py
    
    from cgkit.noise import *
    
    def proc2(P, F, **keyargs):
        r = 0.05
        RiTransformBegin()
        RiConcatTransform(F)
        RiDisk(r*snoise(5*P), r, 360)
        RiTransformEnd()

Using this procedure results in the following image:

.. image:: pics/torus2.jpg

Here are some more examples of what can be generated with appropriate
procedure functions.

Grass:

.. image:: pics/grass2.jpg

Hair (grown on the Stanford Bunny):

.. image:: pics/hairy_bunny.jpg

Gaseous stuff emanating from the Utah teapot:

.. image:: pics/teapot_nebula.jpg

In the first two examples the procedure was generating curves directly
on the surface of an object and in the last example it was generating
points that were displaced from the surface. When you also use curves
or points it is not recommended to stick each curve or point into a
separate RiCurves() or RiPoints() primitive as this might slow down
the rendering process considerably. Instead your procedure should gather
a larger amount of curves/points and use one RiCurves()/RiPoints()
primitive for all of them.
