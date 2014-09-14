More on Soft Bodies
===================

In Bullet, Soft bodies offer many features and possible configurations. Unfortunately, they are poorly documented, but this chapter constitutes a good starting point to learn about them.

Config
------

### Gravity

Soft bodies gravity is set separately from the rigid bodies one. It is apparently a design problem and [may be fixed][separate-gravity-issue] in a future rewrite of Bullet's soft bodies.

	btVector3 gravity(0, -10, 0);

	// This line only affects RIGID bodies gravity
	dynamicsWorld->setGravity(gravity);

	// This one affects SOFT bodies gravity
	dynamicsWorld->getWorldInfo().m_gravity = gravity;

### Cluster

A *cluster* consists of a group of nodes that reacts to collisions together. Using clusters for collisions would improve performance, but soft bodies would behave more like rigid bodies. The syntax is:

	softBody->generateCluster(k);

With `k` the number of clusters to generate. If `k` = 0, the method will generate a cluster for each tetrahedron or triangle.

This method is required if you use soft body joints.


### Configuration properties

Soft bodies have many properties that can be changed, mostly in their `m_cfg` attribute.

	softbody->m_cfg.kDF = 1; // set the dynamic friction

Here is a list from *btSoftBody.h* (again):

	/* Config		*/ 
	struct	Config
	{
		eAeroModel::_			aeromodel;		// Aerodynamic model (default: V_Point)
		btScalar				kVCF;			// Velocities correction factor (Baumgarte)
		btScalar				kDP;			// Damping coefficient [0,1]
		btScalar				kDG;			// Drag coefficient [0,+inf]
		btScalar				kLF;			// Lift coefficient [0,+inf]
		btScalar				kPR;			// Pressure coefficient [-inf,+inf]
		btScalar				kVC;			// Volume conversation coefficient [0,+inf]
		btScalar				kDF;			// Dynamic friction coefficient [0,1]
		btScalar				kMT;			// Pose matching coefficient [0,1]		
		btScalar				kCHR;			// Rigid contacts hardness [0,1]
		btScalar				kKHR;			// Kinetic contacts hardness [0,1]
		btScalar				kSHR;			// Soft contacts hardness [0,1]
		btScalar				kAHR;			// Anchors hardness [0,1]
		btScalar				kSRHR_CL;		// Soft vs rigid hardness [0,1] (cluster only)
		btScalar				kSKHR_CL;		// Soft vs kinetic hardness [0,1] (cluster only)
		btScalar				kSSHR_CL;		// Soft vs soft hardness [0,1] (cluster only)
		btScalar				kSR_SPLT_CL;	// Soft vs rigid impulse split [0,1] (cluster only)
		btScalar				kSK_SPLT_CL;	// Soft vs rigid impulse split [0,1] (cluster only)
		btScalar				kSS_SPLT_CL;	// Soft vs rigid impulse split [0,1] (cluster only)
		btScalar				maxvolume;		// Maximum volume ratio for pose
		btScalar				timescale;		// Time scale
		int						viterations;	// Velocities solver iterations
		int						piterations;	// Positions solver iterations
		int						diterations;	// Drift solver iterations
		int						citerations;	// Cluster solver iterations
		int						collisions;		// Collisions flags
		tVSolverArray			m_vsequence;	// Velocity solvers sequence
		tPSolverArray			m_psequence;	// Position solvers sequence
		tPSolverArray			m_dsequence;	// Drift solvers sequence
	};

Or you can get [this pdf file][soft-body-properties] posted on the Bullet forums by *dphil* for more details.


### Collision flags

Bullet allow multiple configurations for soft body collisions. You can set it via the `softBody->m_cfg.collisions` attribute.

The available values are (from *btSoftBody.h*):

	///fCollision
	struct fCollision { enum _ {
		RVSmask	=	0x000f,	///Rigid versus soft mask
		SDF_RS	=	0x0001,	///SDF based rigid vs soft
		CL_RS	=	0x0002, ///Cluster vs convex rigid vs soft

		SVSmask	=	0x0030,	///Rigid versus soft mask		
		VF_SS	=	0x0010,	///Vertex vs face soft vs soft handling
		CL_SS	=	0x0020, ///Cluster vs cluster soft vs soft handling
		CL_SELF =	0x0040, ///Cluster soft body self collision
		/* presets	*/ 
		Default	=	SDF_RS,
		END
	};};

According to these lines of code, the default is `btSoftBody::fCollision::SDF_RS`. The `CL_*` flags involve clusters, thus if you want to use them, you must call `generateClusters()` in order to see any effect. The following line demonstrates how to set collisions flags.

	softBody->m_cfg.collisions = btSoftBody::fCollision::CL_SS + btSoftBody::fCollision::CL_RS;


### Collisions problems

If you use the flag `btSoftBody::fCollision::CL_RS` on a soft body, it would look like a rigid body on contact to rigid bodies. To avoid that, use `btSoftBody::fCollision::SDF_RS` instead, which is the default setting. But another problem is that rigid bodies may pass through its faces. A workaround is to set the density of the soft body to a higher value. For example:

	softBody->setVolumeDensity(10);

However, since density = mass / volume, the side-effect of this method is the modification of the mass of the body.

### Joints

In Bullet, *joints* are the equivalent of rigid body *constraints* for soft bodies. But contrary to constraints, joints allow us to link soft bodies together, or between a soft and a rigid body. There are 2 types of joints:

* Linear joints (`LJoint`)
* Angular joints (`AJoint`)

Linear joints link two bodies to a fixed distance. To add one, create a `btSoftBody::LJoint::Specs`, set the `position` property and use the `appendLinearJoint()` method of a soft body.

	btSoftBody::LJoint::Specs lJoint;
	lJoint.position = btVector3(0, 5, 0);
	softBody->appendLinearJoint(lJoint, rigiBody);

![Example of linear joint][linear-joint]

Angular joints link two bodies to the same orientation around an axis. The axis is set via the `axis` property.

	btSoftBody::AJoint::Specs aJoint;
	aJoint.axis = btVector3(0, 0, 1);
	softBody->appendAngularJoint(aJoint, rigidBody);

![Example of angular joint][angular-joint]

Using these joints together, it is possible to tightly attach a soft body to a rigid body.

![][bunny-with-sphere]

To remove a joint, get the `btSoftBody::Joint` object from the `m_joints` attribute of the soft body. Then set `m_delete` to `true`.

	joint = softbody->m_joints[softbody->m_joints.size()-1];
	joint->m_delete = true;

It will be marked and be removed automatically after the next simulation step.

### Applying force

It is possible to apply a force to a soft body with the method `addForce()`. From 

	/* Add force (or gravity) to the entire body							*/ 
	void				addForce(		const btVector3& force);
	/* Add force (or gravity) to a node of the body							*/ 
	void				addForce(		const btVector3& force, int node);

For example, the following apply a force of 100000 units in the X axis direction:

	btVector3 direction(1, 0, 0);
	softBody->addForce(direction * 100000);


[linear-joint]: img/soft-more/01_linear_joint.png
[angular-joint]: img/soft-more/02_angular_joint.png
[bunny-with-sphere]: img/soft-more/03_bunny-with-sphere.png

[separate-gravity-issue]: http://code.google.com/p/bullet/issues/detail?id=505
[soft-body-properties]: http://bulletphysics.org/Bullet/phpBB3/viewtopic.php?p=24280#p24280

