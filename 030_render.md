Rendering
=========

Bullet does not have any rendering capabilities, since it is only physics engine. All this task is up to the user/developer.

For the windowing toolkit, which provides an OpenGl context, you can use GLUT which is old or other libraries such as:

* SDL
* SFML
* Qt

SFML
----

http://www.sfml-dev.org/download.php

include:

*&lt;sfml directory>/include/*

libraries:

*directory: &lt;sfml directory>/lib/*
libraries (libname[-s][-d], s for static, d for debug):

* sfml-graphics-d
* sfml-window-d
* sfml-system-d

For more informations, please see this excellent tutorial in SFML official website:
http://www.sfml-dev.org/tutorials/2.1/window-opengl.php

btIDebugDraw Interface
----------------------

Fortunately, Bullet provides an interface called `btIDebugDraw` which helps drawing the basics in order to visualise the physics simulation. 
The user simply indicates how to draw basic shapes, for example a line in OpenGL. 

Create a subclass of `btIDebugDraw`
Implement at least the `drawLine()` method as well as 
reportErrorWarning
draw3dText
setDebugMode
getDebugMode	
