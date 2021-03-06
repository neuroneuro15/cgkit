.. % Hammersley


:mod:`hammersley` --- Generating Hammersley and Halton points
=============================================================

.. module:: cgkit.hammersley
   :synopsis: Generating Hammersley and Halton points


This module contains functions to generate points that are uniformly distributed
and stochastic-looking on either a unit square or a unit sphere. The Hammersley
point set is more uniform but is non-hierarchical, i.e. for different n
arguments you get an entirely new sequence. If you need hierarchical behavior
you can use the Halton point set.

This is a Python version of the implementation provided in:

   | Tien-Tsin Wong, Wai-Shing Luk, Pheng-Ann Heng
   | *Sampling with Hammersley and Halton points* 
   | Journal of Graphics Tools, Vol. 2, No. 2, 1997, pp. 9-24
   | `<http://jgt.akpeters.com/papers/WongLukHeng97/>`_
   | `<http://www.cse.cuhk.edu.hk/ ttwong/papers/udpoint/udpoints.html>`_

.. % planeHammersley


.. function:: planeHammersley(n)

   Yields a sequence of *n* tuples (*x*, *y*) which represent a point on the unit
   square. The sequence of points for a particular *n* is always the same. When *n*
   changes an entirely new sequence will be generated.

   This function uses a base of 2.

.. % sphereHammersley


.. function:: sphereHammersley(n)

   This function yields *n* :class:`vec3` objects representing points on the unit
   sphere. The sequence of points for a particular *n* is always the same. When *n*
   changes an entirely new sequence will be generated.

   This function uses a base of 2.

.. % planeHalton


.. function:: planeHalton(n, p2=3)

   This function yields a sequence of two floats (*x*, *y*) which represent a point
   on the unit square. The number of points to generate is given by *n*. If *n* is
   set to ``None``, an infinite number of points is generated and the caller has to
   make sure the loop stops by checking some other critera.  The sequence of
   generated points is always the same, no matter what *n* is (i.e. the first n
   elements generated by the sequence ``planeHalton(n+1)`` is identical to the
   sequence *planeHalton(n)*).

   This function uses 2 as its first prime base whereas the second prime *p2*
   (which must be a prime number) can be provided by the user.

.. % sphereHalton


.. function:: sphereHalton(n, p2=3)

   This function yields a sequence of :class:`vec3` objects representing points on
   the unit sphere. The number of points to generate is given by *n*. If *n* is set
   to ``None``, an infinite number of points is generated and the caller has to
   make sure the loop stops by checking some other critera. The sequence of
   generated points is always the same, no matter what *n* is (i.e. the first n
   elements generated by the sequence ``sphereHalton(n+1)`` is identical to the
   sequence ``sphereHalton(n)``).

   This function uses 2 as its first prime base whereas the second base *p2* (which
   must be a prime number) can be provided by the user.

.. % ---Copyright---

.. note::

   The original C versions of these functions are distributed under the following
   license:

   (c) Copyright 1997, Tien-Tsin Wong ---  ALL RIGHTS RESERVED ---  Permission to
   use, copy, modify, and distribute this software for any purpose and without fee
   is hereby granted, provided that the above copyright notice appear in all copies
   and that both the copyright notice and this permission notice appear in
   supporting documentation,

   THE MATERIAL EMBODIED ON THIS SOFTWARE IS PROVIDED TO YOU "AS-IS" AND WITHOUT
   WARRANTY OF ANY KIND, EXPRESS, IMPLIED OR OTHERWISE, INCLUDING WITHOUT
   LIMITATION, ANY WARRANTY OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.
   IN NO EVENT SHALL THE AUTHOR BE LIABLE TO YOU OR ANYONE ELSE FOR ANY DIRECT,
   SPECIAL, INCIDENTAL, INDIRECT OR CONSEQUENTIAL DAMAGES OF ANY KIND, OR ANY
   DAMAGES WHATSOEVER, INCLUDING WITHOUT LIMITATION, LOSS OF PROFIT, LOSS OF USE,
   SAVINGS OR REVENUE, OR THE CLAIMS OF THIRD PARTIES, WHETHER OR NOT THE AUTHOR
   HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LOSS, HOWEVER CAUSED AND ON ANY
   THEORY OF LIABILITY, ARISING OUT OF OR IN CONNECTION WITH THE POSSESSION, USE OR
   PERFORMANCE OF THIS SOFTWARE.

