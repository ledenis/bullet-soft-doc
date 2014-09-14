Limitations
===========

Collisions soft vs rigid bodies
-------------------------------

In Bullet, soft bodies are still not perfect. Rigid bodies can penetrate into soft bodies faces, particularly when the rigid bodies are small, even if they are attached together. For example, here is a demonstration of a sphere built with small pieces of rigid bodies.

[![][soft-rigid-issue-img]][soft-rigid-issue-demo]

Fortunately, there exists a workaround. This consists in either using clusters, or increasing the density with `setVolumeDensity()`. See the chapter on soft bodies for more infos.

Scale of the world
------------------

By default, Bullet assumes that all values are in standard units, such as meters, seconds or kilograms. If your application simulate a bigger or smaller than a meter-sized world, you can scale the world to use differents units. See the [wiki][scaling-wiki] to learn how to do so.

However, there seems to exist some problems. At a small scale, like in centimeter units, objects may experience jitter. The same problem appears with a high gravity. The video shows a cube on a plane under a gravity of 1000 units (or m/s²).

[![][high-gravity-jitter-img]][high-gravity-jitter-demo]

[soft-rigid-issue-demo]: https://www.dropbox.com/s/5bth7sj4heob4pi/soft-rigid-collision-issue.avi?dl=0
[high-gravity-jitter-demo]: https://www.dropbox.com/s/r3iqkfktfb2e1qd/high-gravity-jittering.avi?dl=0

[soft-rigid-issue-img]: img/limit/01_soft-rigid-collision-issue.png
[high-gravity-jitter-img]: img/limit/02_high-gravity-jitter.png

[scaling-wiki]: http://www.bulletphysics.org/mediawiki-1.5.8/index.php?title=Scaling_The_World