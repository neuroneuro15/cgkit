First Steps: An introductory tutorial
=====================================

This tutorial gives a short introduction in the usage of the cgkit
package. In the first example, you create a simple scene that just has
one sphere (sort of a "Hello World" scene). To do so, create a file
called "helloworld.py" that contains the following line::

  Sphere()

Now launch the viewer tool passing the above file as argument (if you
have downloaded the source package, don't invoke the viewer tool from
inside the cgkit directory. If you do, Python will load the cgkit package
from the wrong directory and you'll get an ImportError exception)::

  > viewer.py helloworld.py

The result should look something like this:

.. image:: pics/helloworld.jpg

The viewer tool reads the contents of the file which in this case is
an ordinary Python file and displays the scene using OpenGL. When the
input file is processed via the viewer tool it is executed in a
special environment where a couple of modules have already been
imported. That's why calling ``Sphere()`` doesn't result in a
``NameError`` exception. If you import the relevant modules yourself
you can also call the script without the viewer tool (however, you
wouldn't get a visualization of the scene then). You can also create
the above scene directly in a Python shell::

  >>> from cgkit.all import *
  >>> Sphere()
  <cgkit.quadrics.Sphere object at 0x051CC2D0>

The first line imports all you need from cgkit which has to be done
manually now. The second line creates an instance of the ``Sphere``
class. Usually, each object automatically inserts itself into the
scene, so we don't have to keep the resulting reference. Now let's
create another object::

  >>> b=Box(name="Cube", pos=(1.5,2,0))
  >>> listWorld()
  Root
  +---Cube (Box/BoxGeom)
  +---Sphere (Sphere/SphereGeom)

The first line creates a box object. This time we are passing a couple
of parameters like the object's name and its position and we store the
object in the variable *b* so we can manipulate the box afterwards. The
second line calls the ``listWorld()`` function which
prints a tree representation of the current scene. Now it's time for a
little nitpicking, actually the function only displays the *world*
(hence its name) and not the entire *scene*. The world is what you see,
it stores all 3D objects that have a visual representation and is part
of the scene. The whole scene also contains other objects such as the
timer, animation curves, etc. An object stored in the scene is called
a *component* and an object stored in the world is, well, a *world object*
(which is also a component as it is also part of the scene). But back
to the example. We have kept a reference to the box, so let's see what
we can do with it::

  >>> b.name = "The Cube"
  >>> listWorld()
  Root
  +---Sphere (Sphere/SphereGeom)
  +---The Cube (Box/BoxGeom)
  >>> b.pos
  (1.5, 2, 0)
  >>> b.pos=vec3(1,0,2)
  >>> b.pos
  (1, 0, 2)
  >>> b.scale
  (1, 1, 1)

Every world object has a set of attributes that defines its state. The
exact set of attributes depends on the type of object, but there are
some common attributes that every world object has such as a name or a
position (see the cgkit manual for a reference of the available world
objects together with their attributes).

In the first example, we were only specifying one sphere with its
default attributes, that's why we had some geometry in the scene. But
for a 3D scene to be displayed you usually need two more ingredients:
a camera and some light. In the above case, a default camera and light
source was created by the viewer tool. In the following example, we
specify a complete scene, including a camera, two colored light
sources and a sphere with a material assigned to it. Create a file
"simplescene.py" with the following content::

  TargetCamera(
      pos    = (3,2,2),
      target = (0,0,0)
  )

  GLPointLight(
      pos       = (3, -1, 2),
      diffuse   = (1, 0.7, 0.2)
  )

  GLPointLight(
      pos       = (-5, 3, 0),
      diffuse   = (0.2, 0.2, 0.5),
      intensity = 3.0
  )

  Sphere(
      name      = "My Sphere",
      radius    = 1.0,
      material  = GLMaterial(
                     diffuse = (0.7, 1, 0.7)
                  )
  )

Display the scene by calling::

  > viewer.py simplescene.py

The result is this:

.. image:: pics/simplescene1.jpg

By default, you can navigate in the scene using the ``Alt`` key in
combination with the three mouse buttons (if you reach a pole the
camera position will jump around. This is because we are using a
TargetCamera that always tries to align its local "up" direction with
the global "up" direction, so this type of camera can't be "upside
down").

If you have a RenderMan renderer installed (there are free ones
available such as 3Delight_, Aqsis_ or Pixie_) you can try to visualize
the above scene with a different tool::

  > render.py -r<renderer> simplescene.py

``<renderer>`` has to be replaced with the renderer you are using
(e.g. ``3delight``, ``aqsis``, ``pixie``, ``prman``, etc.). This tool will
display the same scene, but this time not using OpenGL but the
specified renderer. The result looks similar than before but is much
smoother:

.. image:: pics/simplescene2.jpg

So if you want to create photorealistic images you can use the viewer
tool for previews and the render tool for creating the final image.


.. _Aqsis: http://aqsis.sf.net/
.. _Pixie: http://pixie.sf.net/
.. _3Delight: http://www.3delight.com/

