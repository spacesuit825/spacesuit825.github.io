# Motion Integration

stardust integrates particle positions in time based on the accelerations derived from the forces computed in the Contact Resolution phase of the simulation pipeline. 

## Position

Position is integrated using the semi-implicit Euler method (Euler-Cromer method). This method is 1st-order accurate in time.

It is a popular choice in DEM codes and is symplectic. It is very simple to implement and provides much of the benefit of an implicit scheme with significantly less complexity.

The use of semi-implicit Euler with velocity-based contact systems is also simpler and can use velocity directly unlike other integration methods.

## Orientation

Particle orientation is integrated using the SPIRAL method developed by del Valle, et al. It is 3rd-order accurate in time.

Angular velocity is integrated using the Strong Stability Preserving Runge Kutta 3 method (SSPRK3). stardust assumes the torque on a particle is constant over a timestep to avoid recomputing forces.

Per the del Valle paper, we first update the quaternion then the angular velocity. 

SPIRAL and SSPRK3 are popular choices for quaternion and angular velocity integration in open-source DEM codes and offer exceptional stability. They preserve the norm of the quaternion and therefore do not require renormalisation of the quaternion each step.