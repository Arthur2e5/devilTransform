devilTransform
==============

Help you trade with GCJ-84 and BD-09 devils while maintaining your WGS-84 sanity.


What are these evil coordinate systems?
---------------------------------------

GCJ-02 is a coordinate system required for all electronic maps for data. It features
a non-linear derivation , or *encryption*, with a maximum of 700 meters, to aid with
national security. The monotonicity is preserved.

From leaked code in [on4wp7] (derived from [emq]?), we can figure out about the one-way
obfuscation

TODO: filter out the obfuscation part from the transformation part.

~ MathDefs
\newcommand\glat{{\rm obfsLatWGS}}
\newcommand\glng{{\rm obfsLngWGS}}
\newcommand\wlat{{\rm weirdLat}}
\newcommand\wlng{{\rm weirdLng}}
~ 

$$
\glat (degLat, degLng) = 
$$

$$
\glng (degLat, degLng) = 
$$

The $(105, 35)$ magic numbers give the mean lat-lng of China.

This pair of values are then transformed to the [Krasovsky 1940][sk42] Ellipsoid used
by old Chinese measurements like Beijing 1954. Other remarkable attempts to solve this
problem includes [wu2010]'s regression.

The BD-09 system adds (apparently) very crude offsets to GCJ-02 coords, illustrated by
[coollpf2013bd] and defined as such:

1. Let $k$ be ${\rm hypot}(degLat, degLng) + 0.00002 \times \sin(radLng / 3000)$;
2. Let $\theta$ be $\arctan(degLng, degLat) + 0.000003 \times \cos(radLat / 3000)$;
3. Return coordinate $bdDegs$ such that:
  - $bdDegLat = k \times \cos(\theta) + 0.0065$;
  - $bdDegLng = k \times \sin(\theta) + 0.0060$;

Like its predecessor, BD-09 maintains a not-too-far-but-enough-to-screw-you-up offset
and good monotonicity. This can be used to obtain reverse approximations with a
relatively small error tolerance in a generally quick way:

1. Let $x$ and $y$ be the $degLat$ and $degLng$ components of the input $weirdCoord$
  element respectively.
2. Let $t$ be $1$.
3. Repeat while $t < t_{max} \land (\abs{\Delta x} > tol \lor \abs{\Delta y} > tol)$:
  i. Let $(x_0, y_0)$ be $(x,y)$.
  2. Let $x$ be $\wlat(x, y)$.
  3. Let $y$ be $\wlng(x, y)$.
  4. Let $(\Delta x, \Delta y)$ be $(x_0 - x, y_0 - y)$.
  4. Increment $t$ by $1$.
4. Return $(x, y)$.

Other reverse methods including negatively applying the delta from the transformation
only once, using the assumption that the offset is not going to cause huge shifts in deltas. This has been reported to give 2-meter-level accuracy for GCJ-to-WGS conversion.

  [coollpf2013bd]: http://blog.csdn.net/coolypf/article/details/8569813
  [wu2010]: https://wuyongzheng.github.io/china-map-deviation/paper.html
  [sk42]: https://en.wikipedia.org/wiki/SK-42_reference_system
  [emq]: https://emq.googlecode.com/svn/emq/src/Algorithm/Coords/Converter.java
  [on4wp7]: https://on4wp7.codeplex.com/SourceControl/changeset/view/21483#353936
