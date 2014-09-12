Introduction
============
![Bullet logo][bullet-logo]

This document is a documentation of the physics engine Bullet Physics. It aims at introducing new users the basics of this engine such as rigid and soft body simulation as well as additional techniques such as Voronoi shattering. In complement, an example implementation  is provided to demonstrate and illustrate the content of this documentation.

Context
--------

### What is Bullet?

Bullet Physics is a open source 3D physics engine. It has been used in notable products such as commercial video games, movies or 3D programs. Created by Erwin Coumans, Bullet Physics is able to simulate collision detection, rigid and soft body dynamics. It is officially distributed as a C++ library, but other fan-made implementations exist. You can find more informations on its website: <http://bulletphysics.org/>

At the time of writing this documentation (September 2014), the last stable release of Bullet is the version 2.82 (hosted on [Google Code][bullet-2-release]), while the version 3 (available on [GitHub][bullet-3]) is currently in development.


### Why this documentation?

Although Bullet Physics is a popular physics library, its [official documentation][bullet-wiki] is still lacking, especially regarding the use of soft bodies. Some informations have to be collected from its forums, demos, source code or from other sources. The goal of this documentation is to fill this gap.

Implementation
-------------

To demonstrate the differents points discussed in this documentation, an implementation is provided, as source code and Windows executable.

The scenario consists in a sphere on a soft (that is not rigid) and rugged ground. Other objects fall from above and strike the sphere which breaks into parts.

Outline
-------

**Using Bullet Physics**

* [Setting Up a Bullet Application][setup]

* [Rendering][render]

* [Rigid Bodies][rigid]

* [Basics of Soft Bodies][soft]

* [More on Soft Bodies][soft-more]


**Additional concepts**

* [Importing from Blender as tetrahedral Soft Bodies][import]

* [Voronoi Fracture and Destruction][voronoi]

* [Generating a Movie][movie]


[bullet-logo]: img/bullet-logo.png

[bullet-3]: https://github.com/bulletphysics/bullet3
[bullet-2-release]: https://code.google.com/p/bullet/downloads/list
[bullet-wiki]: http://bulletphysics.org/mediawiki-1.5.8



[setup]: 010_setup.html
[render]: 030_render.html
[rigid]: 021_rigid.html
[soft]: 040_soft.html
[soft-more]: 041_soft_more.html

[import]: 055_import.html
[voronoi]: 060_voronoi.html
[movie]: 080_movie.html
