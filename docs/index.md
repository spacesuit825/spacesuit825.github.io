# stardust-dem

## About stardust-dem

stardust-dem is free to use but closed-source (refer License).

stardust-dem (or just stardust) is a DEM (Discrete Element Method) solver. \ 
The current version is the 5th iteration of the engine and has taken a long time to \
reach this level of maturity. stardust-dem provides best-in-class performance and \
can match or beat comparable open-source and commerical GPU DEM solvers for most common \
usecases (sometimes by up to 5x).

stardust-dem only supports NVIDIA GPUs at this time. There is ongoing work to enable \
AMD ROCm HIP support. 

stardust-dem only supports single and multi-spheres at this time. Support for \
arbitrary polyhedra is planned.

stardust is a capable DEM solver with a lot of the features expected in industry-grade \
solvers. Features implemented by commerical solvers that are not yet supported likely will be.

Some of the features stardust-dem supports:
* Simulation of single and multi-sphere granular particles of arbitrary complexity/size distribution
* Generation of particles into complex (mesh-based) containers
* Complex prescribed mesh motions via motion chaining
* Simulation of conveyors via surface motions
* Stopping and starting simulations via checkpoints
* Deterministic results, if desired (at some runtime performance penalty)
* Acceleration of massive scale simulations via active regions
* Execution on both Windows and Linux 
* Support for single and double precision
* Running with or without the GUI

In addition, stardust-dem provides a comprehensive and easy-to-use plugin system which enables \
almost any behaviour in the engine to be modified. A few examples of possible plugin usecases are: \
* Custom contact models, including bonding and wear
* Deformable/rigid body meshes
* Custom behaviour in specific areas of the simulation domain
* Custom particle properties, such as moisture, temperature or particle tracking
* Coupling with fluid solvers
* Coupling with rigid-body solvers

The stardust-dem plugin system requires no compiler installation but does require some basic knowledge of \
C/C++ syntax (there is no need to interact with any manual memory management).

Refer to the Plugin System section of the Technical chapter for detailed documentation.


stardust-dem provides a selection of builtin contact models which are implemented via the plugin \
system. The builtin models currently supported are:
* Hertz-Mindlin no-slip + Type C rolling resistance (default)
* ... more when I get to it



In terms of solving and simulation, stardust is stable. The results that it produces are validated \
against existing experimental datasets (see Validation). 

The GUI and the internals of the engine are still heavily in beta. Expect performance, feature \
and quality-of-life improvements.

## Use Cases 

stardust-dem can simulate almost any industrial granular problem and is capable of doing so quickly. 

Some example simulations can be seen below.
