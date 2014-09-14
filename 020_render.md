Rendering
=========

Bullet does not have any rendering capabilities. Although it has some interfaces such as `btIDebugDraw` that facilitate its implementation, Bullet remains mostly a physics engine. All this task is up to the user developer.

To draw in 3D, *OpenGL* (Open Graphics Library), a graphics programming interface, can be used. It is commonly used to draw 3D scenes and is cross platform. However, this chapter does not explain how to use OpenGL, but rather gives some clues to use it with Bullet. If you don't know OpenGL, you can find lots of tutorials that exist on the internet. For example, the popular but old [Nehe tutorials][nehe-tuto] for OpenGL 2. For OpenGL 3 or later, [*Learning Modern 3D Graphics Programming*][arcsynthesis-tuto] is a good one. Though it is old and lots of features are deprecated in next versions, OpenGL 2 is used in the implementation provided with this documentation because of its simplicity in order to focus more on Bullet Physics.

OpenGL, however, cannot by itself draw on the screen. It can only be used within an *OpenGL context*. This is usually provided by a *windowing toolkit*, which implements a window system and also keyboard / mouse interaction.

 Examples of popular windowing toolkit include:

* [SDL][sdl-website] (Simple DirectMedia Layer): a C library
* [SFML][sfml-website] (Simple and Fast Multimedia Library): a C++ library
* [Qt][qt-website]: a C++ framework, which is basically used to build user interfaces windows (with buttons, text fields...)

This chapter focuses on the SFML library because it is simple to use and is C++ friendly.

Note: *GLUT* (OpenGL Utility Toolkit) is an other example of well known window toolkit, created for OpenGL, but since it is old and no longer maintained by its author, it is not recommended to use it.

SFML
----

SFML can be found on its [download page on its website][sfml-download]. The package releases are ready to use, and do not need to be compiled.

Here is the configuration to set for a C++ project that uses the library.

**include:**
*&lt;sfml directory>/include/*

**lib directory:**
*&lt;sfml directory>/lib/*
**libraries:**
(libname[-s][-d], s for static, d for debug):

* sfml-graphics-d
* sfml-window-d
* sfml-system-d

Please see the [tutorials page on its website][sfml-tuto], *Getting started* section for more informations. Especially, to use OpenGL within an window in SFML, please see this [tutorial][sfml-opengl-tuto].


btIDebugDraw Interface
----------------------

Fortunately, Bullet provides an interface called `btIDebugDraw` which helps drawing the basics in order to visualise the physics simulation. 
As its name suggests, it is suitable for debugging purpose, or when a more optimised solution has yet to be implemented into the application. The user simply indicates how to draw basic shapes, for example a line in OpenGL.

However, the modern OpenGL (version >= 3) approach does not work well with this interface which requires the OpenGL *immediate mode* (with `glBegin()` and `glEnd()`). Indeed, OpenGL version 3 (or later) deprecated this mode in favor of a lower level approach with shader 

The steps to use `btDebugDraw` are as follows:

1. Create a subclass of `btIDebugDraw`
2. Implement at least these 6 *virtual pure functions* (but can be left empty):
	* `void	drawLine(const btVector3& from,const btVector3& to,const btVector3& color)`
	* `void drawContactPoint(const btVector3& PointOnB,const btVector3& normalOnB,btScalar distance,int lifeTime,const btVector3& color)`
	* `void reportErrorWarning(const char* warningString)`
	* `void	draw3dText(const btVector3& location,const char* textString)`
	* `void	setDebugMode(int debugMode)`
	* `int getDebugMode() const`
3. Set this debug drawer to the Bullet world via *setDebugDrawer()*:
		`dynamicsWorld->setDebugDrawer(debugDraw);`

Here is a minimal example of a class that implement it (*#include* directives omitted):

*In DebugDraw.h:*

	class DebugDraw: public btIDebugDraw {
	public:
		virtual void drawLine(const btVector3& from, const btVector3& to,
				const btVector3& color);
		virtual void drawContactPoint(const btVector3& PointOnB,
				const btVector3& normalOnB, btScalar distance, int lifeTime,
				const btVector3& color);

		virtual void reportErrorWarning(const char* warningString);

		virtual void draw3dText(const btVector3& location, const char* textString);

		virtual void setDebugMode(int debugMode);

		virtual int getDebugMode() const;
	};

*In DebugDraw.cpp:*

	void DebugDraw::drawLine(const btVector3& from, const btVector3& to,
			const btVector3& color) {
		glPushMatrix();

		glColor3f(color.m_floats[0], color.m_floats[1], color.m_floats[2]);
		glBegin(GL_LINES);
		glVertex3f(from.m_floats[0], from.m_floats[1], from.m_floats[2]);
		glVertex3f(to.m_floats[0], to.m_floats[1], to.m_floats[2]);
		glEnd();

		glPopMatrix();
	}

	void DebugDraw::drawContactPoint(const btVector3& PointOnB,
			const btVector3& normalOnB, btScalar distance, int lifeTime,
			const btVector3& color) {
	}

	void DebugDraw::reportErrorWarning(const char* warningString) {
	}

	void DebugDraw::draw3dText(const btVector3& location, const char* textString) {
	}

	void DebugDraw::setDebugMode(int debugMode) {
	}

	int DebugDraw::getDebugMode() const {
		return DBG_DrawWireframe;
	}

`drawLine()` is the method that calls the drawing functions. `getDebugMode()` always returns the value `DBG_DrawWireframe`, which means to be the debug drawer always draw as wireframes. Despite the fact that the 4 other functions are not used, they must be implemented. Here is what a sphere may look like when rendered via the debug drawer:

![][sphere]

[sphere]: img/render/01_sphere.png

[nehe-tuto]: http://nehe.gamedev.net/
[arcsynthesis-tuto]: http://www.arcsynthesis.org/gltut/

[sdl-website]: https://www.libsdl.org/
[sfml-website]: http://www.sfml-dev.org/
[qt-website]: http://qt-project.org/

[sfml-download]: http://www.sfml-dev.org/download.php
[sfml-tuto]: http://sfml-dev.org/tutorials/2.1/
[sfml-opengl-tuto]: http://www.sfml-dev.org/tutorials/2.1/window-opengl.php