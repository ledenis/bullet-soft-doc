Limitations
===========

Collisions soft vs rigid bodies
-------------------------------

In Bullet, soft bodies are still not perfect. Rigid bodies can penetrate into soft bodies faces, particularly when the rigid bodies are small. Fortunately, there exists a workaround. This consists in either using clusters, or increase the density with `setVolumeDensity()`. See the chapter on soft bodies for more infos.

Scale of the world
------------------

By default, Bullet assumes that all values are in standard units, such as meters, seconds or kilograms. If your application simulate a bigger or smaller than a meter-sized world, you can scale the world to use differents units. See the [wiki][scaling-wiki] to learn how to do so.

However, there seems to exist some problems. At a small scale, like in centimeter units, objects may experience jitter. The same problem appears with a high gravity.

[scaling-wiki]: http://www.bulletphysics.org/mediawiki-1.5.8/index.php?title=Scaling_The_World