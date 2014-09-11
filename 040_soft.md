Basics of Soft Bodies
=====================

Introduction
------------

### Types of meshes ###

A 3D model can be represented by:

* A triangle mesh, or
* A tetrahedral mesh

In soft body dynamics, a **triangle mesh** is suitable for clothes since it models only the surface. A **tetrahedral mesh**, however, has a notion of volume. Indeed, the mesh is composed of tetrahedrons.

In this chapter, we will see how to create triangle mesh soft bodies. For tetrahedral mesh ones, see the corresponding chapter.

Changing the world
------------------

In order to use soft bodies in Bullet, you must change few lines in the code in comparison to an only-rigid-body simulation. Of course, you will still be able to use rigid bodies.

Soft bodies in Bullet require a special type of world, the `btSoftRigidDynamicsWorld`. In addition, it must take `btSoftBodyRigidBodyCollisionConfiguration` and `btDefaultSoftBodySolver` objects.

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

Do not forget to `delete softSolver`:

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


Soft bodies
-----------

Now we can add soft bodies to this world. To create soft bodies, we can either:

* build the soft body manually by instantiating it, then using methods such as `appendLink()` and `appendFace()` to construct the mesh.
* or use a helper class that Bullet provides called `btSoftBodyHelper`. This class mainly consists of a set of static methods intended to create soft bodies.

We will naturally use the helper class here. For example, we can use it to create a cloth and a ball as **triangle mesh** softbodies.

To create a cloth, the class provides this method:
	
	static btSoftBody* CreatePatch (btSoftBodyWorldInfo& worldInfo, const btVector3& corner00, const btVector3& corner10, const btVector3& corner01, const btVector3& corner11, int resx, int resy, int fixeds, bool gendiags)

with the following arguments:

* *worldInfo*: the object returned from `dynamicsWorld->getWorldInfo()`
* *corner00*, *corner10*, *corner01* and *corner11*: coordinates of the four corners, see below
* *resx* and *resy*: horizontal and vertical resolution
* *fixeds*: integer that indicates which node to fix, see below
* *gendiags*: whether to generate diagonal links

And here is the explanation of the *cornerxx* and *fixeds* parameters (directly taken from *btSoftBodyHelpers.cpp*):

	*  corners:
	*
	*  [0][0]     corner00 ------- corner01   [resx][0]
	*                |                |
	*                |                |
	*  [0][resy]  corner10 -------- corner11  [resx][resy]
	*
	*
	*   "fixedgs" map:
	*
	*  corner00     -->   +1
	*  corner01     -->   +2
	*  corner10     -->   +4
	*  corner11     -->   +8
	*  upper middle -->  +16
	*  left middle  -->  +32
	*  right middle -->  +64
	*  lower middle --> +128
	*  center       --> +256


For example, 

	//TODO

Like rigid bodies, you need to add it to the world:

	dynamicsWorld->addSoftBody(cloth);

...

	cloth->generateBendingConstraints(2);

For a ball, it is simpler:

	static btSoftBody* CreateEllipsoid(btSoftBodyWorldInfo& worldInfo, const btVector3& center, const btVector3& radius, int res)



