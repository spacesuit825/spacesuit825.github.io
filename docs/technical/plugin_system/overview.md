# Plugin System

## Overview

stardust-dem enables the user to execute custom code at specified points within a simulation. These
points are called "hooks". 

stardust does not require an installation of a compiler or any SDK. stardust uses runtime compilation
to generate the required source code at execution time. stardust makes available for the API
headers, however, these are not required for compilation and are instead provided only for Intellisense or IDE support.

## Hooks

stardust-dem makes the following hooks available:

* At simulation initialisation:
  * STARDUST_PLUGIN_INITIALISE_SIMULATION
  * STARDUST_PLUGIN_INITIALISE_GEOMETRIES
  * STARDUST_PLUGIN_INITIALISE_TRIANGLES
  * STARDUST_PLUGIN_INITIALISE_MATERIALS
  * STARDUST_PLUGIN_INITIALISE_MATERIAL_INTERACTIONS
  * STARDUST_PLUGIN_COMPUTE_TIMESTEP

* During the simulation loop:
  * STARDUST_PLUGIN_BEGIN_ITERATION
  * STARDUST_PLUGIN_INITIALISE_PARTICLES (executed on every newly generated particle)
  * STARDUST_PLUGIN_INTEGRATE_GEOMETRIES
  * STARDUST_PLUGIN_BEFORE_CONTACT_FORCE
  * STARDUST_PLUGIN_NORMAL_FORCE_CONTACT_RESPONSE
  * STARDUST_PLUGIN_ADHESIVE_FORCE_CONTACT_RESPONSE
  * STARDUST_PLUGIN_TANGENTIAL_FORCE_CONTACT_RESPONSE
  * STARDUST_PLUGIN_ROLLING_TORQUE_CONTACT_RESPONSE
  * STARDUST_PLUGIN_IMPACT_ENERGY_CONTACT_RESPONSE
  * STARDUST_PLUGIN_AFTER_CONTACT_FORCE
  * STARDUST_PLUGIN_EXTERNAL_FORCE
  * STARDUST_PLUGIN_BEFORE_PARTICLE_INTEGRATION
  * STARDUST_PLUGIN_AFTER_PARTICLE_INTEGRATION
  * STARDUST_PLUGIN_END_ITERATION

The hooks along with all plugin API declarations are provided in "api_common.hpp". 

## Implementing a hook

stardust hook declarations are macros which carry some additional information used by the engine. To implement a hook, the user should write as it the hook declaration is a function definition. Every hook takes the same arguments, which are all API wrapper structs. For example, to implement `STARDUST_PLUGIN_NORMAL_FORCE` the following defintion is required:

    STARDUST_PLUGIN_NORMAL_FORCE_CONTACT_RESPONSE(
        api_v1::SimulationData& simulation,
        api_v1::DiscreteElement& element_1,
        api_v1::DiscreteElement& element_2,
        api_v1::ContactMaterialInteraction& interaction,
        api_v1::ContactData& contact_data,
        api_v1::ContactResponse& response
    ) {

        ... some implementation ...

    }

## Wildcard System

To enable custom user data to persist between timesteps, stardust-dem provides a data storage 
system called wildcards. A wildcard is an arbitrary piece of data that can store some information 
and is automatically tracked and persisted by the solver.

The following wildcard categories exist in stardust-dem:

* Particle wildcards (size: n_particles)
* Geometry wildcards (size: n_geometries)
* Triangle wildcards (size: n_triangles_in_simulation)
* Contact wildcards (size: n_contacts)
* Material wildcards (size: n_materials)
* Material Interaction wildcards (size: n_material_interactions)
* Simulation wildcards (size: 1)

Unlike other solvers, stardust allows any type to be stored inside of a wildcard and allows an arbitrary number of
wildcards per reference (particle, geometry, etc.). Within stardust, these arrays are treated as opaque and are 
not modified in any way by the engine, besides occasional reordering.

Wildcards can be declared in the solver file (with a restriction to only primitive types) or the plugin source file itself. Within the source file wildcards are declared via the following API macros: 

    STARDUST_REQUIRES_PARTICLE_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_GEOMETRY_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_TRIANGLE_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_CONTACT_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_MATERIAL_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_MATERIAL_INTERACTION_WILDCARD(wildcard_name)
    STARDUST_REQUIRES_SIMULATION_WILDCARD(wildcard_name)

A wildcard declaration for a contact wildcard called "tangential_displacement" looks like this in the source file:

    struct ComplexType {
        float3 displacement;
        double time_in_contact;
    };
    
    STARDUST_REQUIRES_CONTACT_WILDCARD(tangential_displacement, ComplexType)

Note that the API macros behave as regular macros in the code and therefore can be disabled via `#ifdef 0` or similar. 

If a wildcard API macro is present in your code and is unguarded, the wildcard will be declared and allocated!

Internally, the wildcards are addressed via an element_index and a wildcard_index. The element is the particle or contact index or similar. The wildcard_index is the index of the wildcard within that particular elements sub-array, i.e., if you declare "tangential_displacement" and then "rolling_torque", the wildcard "tangential_displacement" has a wildcard_index of 0 and "rolling_torque" has an index of 1.

As the engine may declare its own wildcards for builtin models or other purposes, the user should refrain from attempting to rely on bare indexing into the wildcard system. stardust will automatically generate the required indices at runtime. To address the wildcard of interest, such as the one from the previous example "tangential_displacement", the user should use the following tag:

    WC_CONTACT_tangential_displacement

This pattern is the same for other wildcard categories, but of course, for a particle wildcard you should swap out `CONTACT` for `PARTICLE` and so on. Note that the tag is sensitive to the case the wildcard was declared in.

stardust will not provide the raw array address for the wildcards. Access is achieved by the appropriate API wrapper struct. For example, access to the contact wildcard we declared earlier is achieved like so:

    ComplexType wildcard = contact_data.getWildcard<ComplexType>(WC_CONTACT_tangential_displacement);




