Rigid Bodies
============

Rigid bodies are the base of physics simulation. They are undeformable objects and can represent most of the real life objects. Each one needs a collision shape (that can be shared) in order to interact with other bodies. The basics to create a rigid body is explained in the [*hello world* article][hello-wiki2], as well as in this short [article on rigid bodies][rigid-wiki] on the official Bullet wiki.

Constraints
-----------

Rigid bodies can be linked to each other with what Bullet calls **constraints**. The 6 types of constraints are:

* Point-to-Point (Ball socket) (`btPoint2PointConstraint`)
* Hinge (`btHingeConstraint`)
* Slider (`btSliderConstraint`)
* Cone Twist (`btConeTwistConstraint`)
* Generic 6 Degrees of Freedom (DoF) Constraint (`btGeneric6DofConstraint`)
* Fixed (`btFixedConstraint`)

They all implement `btTypedConstraint`. To use one, create a constraint, then call the method `addConstraint()` on the world:

	btFixedConstraint* constraint = new btFixedConstraint(*bodyA, *bodyB, localA, localB);
	dynamicsWorld->addConstraint(constraint, true); // `true` = disable collisions between them

The `setBreakingImpulseThreshold()` allows the release of the link when the impulse applied reaches a threshold.
	
	constraint->setBreakingImpulseThreshold(10);

For more informations, see [this article][constraints-wiki] on the Bullet wiki.

Note: Soft bodies cannot be linked with *constraints*, the equivalent for them is called a **joint**. See the appropriate part of the documentation for details. 

Sticky effect
-------------

Here is a nice effect that can be useful to mimic a sticky object. 

We can simulate this sticky effect between rigid bodies by linking the bodies together on contact, and releasing this link when the force applied between them reaches a certain threshold. To link bodies, we can use **constraints**. To detect collision contacts, we can loop all the contact **manifolds** (pair of colliding bodies).

The manifolds are accessible through the *collision dispatcher* of the world. Once we get it, we can iterating over the manifolds, and extract the 2 colliding bodies. Also make sure that `checkCollideWith()` returns `true`. Otherwise, a link might already exists. A manifold can contain several contacts, for example if a cube is standing on the floor. But in this case, we only process one. In addition, we check if the contact is a penetration (distance < 0).

	// Loop over rigid bodies collision contacts
	btDispatcher* dispatcher = m_dynamicsWorld->getDispatcher();
	int numManifolds = dispatcher->getNumManifolds();
	for (int i=0;i<numManifolds;i++)
	{
		btPersistentManifold* contactManifold =  dispatcher->getManifoldByIndexInternal(i);

		btCollisionObject* obA = const_cast<btCollisionObject*>(contactManifold->getBody0());
		btCollisionObject* obB = const_cast<btCollisionObject*>(contactManifold->getBody1());


		// We want only rigid body contacts
		btRigidBody* rigidObA = btRigidBody::upcast(obA);
		btRigidBody* rigidObB = btRigidBody::upcast(obB);
		if (rigidObA == 0 || rigidObB == 0)
			continue;

		// We check if the collision filters of those two bodies match
		// (esp. the "ignore collision between linked bodies flag")
		if (!rigidObA->checkCollideWith(rigidObB))
			continue;

		int numContacts = contactManifold->getNumContacts();
		for (int j=0;j<numContacts;j++)
		{
			btManifoldPoint& pt = contactManifold->getContactPoint(j);
			if (pt.getDistance()<0.f)
			{

				const btVector3& ptA = pt.getPositionWorldOnA();
				const btVector3& ptB = pt.getPositionWorldOnB();

				// Create constraint between them
				linkRigidBodies(rigidObA, rigidObB, ptA, ptB);

				break; // only process 1 contact

			}
		}
	}

Then we can link the bodies. Here we use btFixedConstraint. The anchor points need to be set to local coordinates. The code is adapted from a [post][link-code-forum] on the forums.

	void linkRigidBodies(btRigidBody* bodyA, btRigidBody* bodyB,
			const btVector3& anchorA, const btVector3& anchorB) {

		float breakingThreshold = 10;

		btTransform localA;
		btTransform localB;

		localA.setIdentity();
		localB.setIdentity();

		// Setting world space origin
		localA.setOrigin(anchorA);
		localB.setOrigin(anchorB);
		// Calculating local space origin
		localA = bodyA->getCenterOfMassTransform().inverse() * localA;
		localB = bodyB->getCenterOfMassTransform().inverse() * localB;

		float totalMass = 1.f/bodyA->getInvMass() + 1.f/bodyB->getInvMass();
		// Creating constraint
		btFixedConstraint* constraint = new btFixedConstraint(*bodyA, *bodyB, localA, localB);
		constraint->setBreakingImpulseThreshold(breakingThreshold * totalMass);
		// Adding constraint
		m_dynamicsWorld->addConstraint(constraint, true);

		bodyA->activate(true);
		bodyB->activate(true);
	}

The breaking of the link is automatically handled by Bullet when using `setBreakingImpulseThreshold()`.

[hello-wiki2]: http://bulletphysics.org/mediawiki-1.5.8/index.php/Hello_World
[rigid-wiki]: http://bulletphysics.org/mediawiki-1.5.8/index.php/Rigid_Bodies
[constraints-wiki]: http://bulletphysics.org/mediawiki-1.5.8/index.php/Constraints
[link-code-forum]: http://www.bulletphysics.org/Bullet/phpBB3/viewtopic.php?f=9&t=8935&view=previous#p30241