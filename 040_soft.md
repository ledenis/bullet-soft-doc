Basics of Soft Bodies
=====================

Introduction
------------

In physics simulation, a *soft body* is, as its name suggests, a deformable object, as opposed to rigid bodies. Soft bodies demand a lot more computation resources than rigid bodies because of their complexity to simulate. Indeed, non only their shapes are changing over time, but their internal structures also need to be considered.

### Types of meshes ###

In general, a 3D model can be represented by:

* A triangle mesh, or
* A tetrahedral mesh

In soft body dynamics, a **triangle mesh** is suitable for clothes since it models only the surface. A **tetrahedral mesh**, however, has a notion of volume. Indeed, this type of mesh is composed of tetrahedrons, which are ultimately the 3D version of triangles. These ones are suitable for soft bodies with a volume.

In this chapter, we will see how to create triangle mesh soft bodies. For tetrahedral mesh ones (which are built with tetrahedrons), see the *Importing from Blender as tetrahedral Soft Bodies* chapter.

Changing the world
------------------

In order to use soft bodies in Bullet, you must change a few lines in the code in comparison to an only-rigid-body simulation. Of course, you will still be able to use rigid bodies.

Soft bodies in Bullet require a special type of *dynamics world*, namely the `btSoftRigidDynamicsWorld`. In addition, it must take `btSoftBodyRigidBodyCollisionConfiguration` and `btDefaultSoftBodySolver` objects as parameters at construction.

Thus, instead of initialising the world with this code:

	btBroadphaseInterface* broadphase = new btDbvtBroadphase();

    btDefaultCollisionConfiguration* collisionConfiguration = new btDefaultCollisionConfiguration();
    btCollisionDispatcher* dispatcher = new btCollisionDispatcher(collisionConfiguration);

    btSequentialImpulseConstraintSolver* solver = new btSequentialImpulseConstraintSolver;

    btDiscreteDynamicsWorld* dynamicsWorld = new btDiscreteDynamicsWorld(dispatcher, broadphase, solver, collisionConfiguration);


We will need to use this code:

	btBroadphaseInterface* broadphase = new btDbvtBroadphase();

	// This line has changed
    btSoftBodyRigidBodyCollisionConfiguration* collisionConfiguration = new btSoftBodyRigidBodyCollisionConfiguration();
    btCollisionDispatcher* dispatcher = new btCollisionDispatcher(collisionConfiguration);

    btSequentialImpulseConstraintSolver* solver = new btSequentialImpulseConstraintSolver;

    // This line has been added
    btSoftBodySolver* softSolver = new btDefaultSoftBodySolver;

    // This line has changed
    btSoftRigidDynamicsWorld* dynamicsWorld = new btSoftRigidDynamicsWorld(dispatcher, broadphase, solver, collisionConfiguration, softSolver);

Do not forget to `delete` the `softSolver`:

	delete dynamicsWorld;
	delete softSolver; // This one
	delete solver;
	delete collisionConfiguration;
	delete dispatcher;
	delete broadphase;

And to add these `#includes` directives (they are not included by btBulletDynamicsCommon.h):

	#include <BulletSoftBody/btDefaultSoftBodySolver.h>
	#include <BulletSoftBody/btSoftBodyRigidBodyCollisionConfiguration.h>
	#include <BulletSoftBody/btSoftRigidDynamicsWorld.h>

A last thing is to add the library *BulletSoftBody* to the build settings of the project. It **must** be placed before *BulletDynamics* in most compilers, due to the dependency order.


Creating Soft bodies
-----------

Now that the world is correctly initialised, we can add soft bodies to it. To create soft bodies, we can either:

* Build the soft body manually by instantiating it, then using methods such as `appendLink()` and `appendFace()` to construct the mesh. Or:
* Use a helper class that Bullet provides called `btSoftBodyHelper`. This class mainly consists of a set of static methods intended to create soft bodies.

We will naturally use the helper class here. For example, we can use it to create a cloth and a sphere as **triangle mesh** soft bodies. For tetrahedral meshes, please see the appropriate chapter.

To create a cloth, the class provides this method:
	
	static btSoftBody* CreatePatch (btSoftBodyWorldInfo& worldInfo, const btVector3& corner00, const btVector3& corner10, const btVector3& corner01, const btVector3& corner11, int resx, int resy, int fixeds, bool gendiags)

which takes the following arguments:

* *worldInfo*: the object returned from `dynamicsWorld->getWorldInfo()`
* *corner00*, *corner10*, *corner01* and *corner11*: coordinates of the four corners, see below
* *resx* and *resy*: horizontal and vertical resolution
* *fixeds*: integer that indicates which node to fix, see below
* *gendiags*: whether to generate diagonal links

There exists also the function `CreatePatchUV()` which takes the same parameters, but supports "UV Texture Coordinates".

And here is the explanation of the *cornerXX* and *fixeds* parameters (taken from the *btSoftBodyHelpers.cpp* source comments with some fixes):

	*  corners:
	*
	*  [0][0]     corner00 ------- corner10   [resx][0]
	*                |                |
	*                |                |
	*  [0][resy]  corner01 -------- corner11  [resx][resy]
	*
	*
	*   "fixeds" map:
	*
	*  corner00     -->   +1
	*  corner10     -->   +2
	*  corner01     -->   +4
	*  corner11     -->   +8

	only with CreatePatchUV():
	*  upper middle -->  +16
	*  left middle  -->  +32
	*  right middle -->  +64
	*  lower middle --> +128
	*  center       --> +256


For example, 

	btSoftBody* cloth = btSoftBodyHelpers::CreatePatchSoftUV(
			btVector3(0, 5, 0),
			btVector3(3, 5, 0),
			btVector3(0, 1, 0),
			btVector3(3, 1, 0),
			10, 10,  1 + 2 + 16 , false);

will create a squared cloth floating above the world origin, anchored by the upper left, upper middle and upper right nodes.

Like rigid bodies, you need to add it to the world:

	dynamicsWorld->addSoftBody(cloth);

To give more resistance to bending, we can add *bending constraints*.

	cloth->generateBendingConstraints(2);

This actually adds links between nodes which are at 2 nodes distance.

Without bending constraints:
![][cloth]

With bending constraints:
![][cloth-bend]

To create a sphere-shaped soft body, the function is simpler:

	static btSoftBody* CreateEllipsoid(btSoftBodyWorldInfo& worldInfo, const btVector3& center, const btVector3& radius, int res)

The class `btSoftBodyHelpers` provides also other functions to create triangle mesh soft bodies from some mesh data (from triangle mesh or from vertices only).

	/* Create from trimesh													*/ 
	static	btSoftBody*		CreateFromTriMesh(	btSoftBodyWorldInfo& worldInfo,
		const btScalar*	vertices,
		const int* triangles,
		int ntriangles,
		bool randomizeConstraints = true);
		
	/* Create from convex-hull												*/ 
	static	btSoftBody*		CreateFromConvexHull(	btSoftBodyWorldInfo& worldInfo,
		const btVector3* vertices,
		int nvertices,
		bool randomizeConstraints = true);

Note that all the previous functions create soft bodies as triangle meshes. The only function that creates a tetrahedral mesh is `CreateFromTetGenData` and is discussed in the *Importing from Blender as tetrahedral Soft Bodies* chapter.

[cloth]: img/soft/01_cloth.png
[cloth-bend]: img/soft/02_cloth-bend.png

