# Plugin System

## Overview

I understand stardust will likely never achieve the inbuilt feature set large commerical solvers naturally have, simply due to the scale and man-hours required. The plugin system built into stardust is my answer to this.

stardust includes a very powerful plugin system which enables a user to customise the behaviour of the engine. The plugin system makes minimal assumptions about the user requirements and have been designed to be a general as possible. Implementation of nearly all commerical DEM program additional features, such as:

* Coupling (fluids and rigid bodies)
* Custom contact models
* Wear models
* Bonding models
* Temperature/heat transfer
* Moisture and evaporative models
* Time-based, position-based or event-based adaptive material/particle/triangle properties
* Particle property tagging/tracking
* Electrostatics
* Coatings/sprays
* Buoyancy/drag/lift models

are possible via the plugin system. As stardust develops the plugin system will be extended to cover more potential use cases.

The plugin system does not require the installation of a compiler or SDK. stardust makes available the API
headers, however, these are provided for Intellisense and IDE support, and not required for compilation.

All stardust builtins are implemented inside of the plugin system.

All stardust owned data is non-dimensionalised to improve floating point behaviour as per the Architecture section. At API boundaries, stardust will automatically convert from its internal unit system to the one you declare in the solver file and vice versa. Your plugin should be written under the unit system you declared in the solver file.

## Important Guidelines

A few important things to note before writing any plugin code:

* Any variable, function, namespace, class or struct with the word `stardust` or `STARDUST` is reserved for internal use. Use of reserved words in any of your names may result in undefined and unpredictable behaviour!


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

stardust hook declarations are macros which carry some additional information used by the engine. To implement a hook, the user should write the hook declaration as if it were a function definition. Every hook takes similar arguments, which are all API wrapper structs. You should refer to the documentation of the specific hook you are implementing to understand its requirements.

For example, to implement `STARDUST_PLUGIN_NORMAL_FORCE_CONTACT_RESPONSE` the following defintion is required:

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

## Plugin Datatypes

Plugins do ntp

## Wildcards and Parameters

Wildcards and Parameters form the backbone of the stardust plugin system and are what make it so versatile. In short:

* Wildcards provide persistent, stateful data storage in the plugin
* Parameters provide data input into the plugin from the host

### Wildcards

A wildcard is a reserved slot for some custom user data. The following wildcard categories exist in stardust-dem:

* Particle wildcards (size: n_particles)
* Particle template wildcards (size: n_templates)
* Geometry wildcards (size: n_geometries)
* Triangle wildcards (size: n_triangles_in_simulation)
* Contact wildcards (size: n_contacts)
* Material wildcards (size: n_materials)
* Material Interaction wildcards (size: n_material_interactions)
* Simulation wildcards (size: 1)

Unlike other solvers, stardust allows data of any type to be stored inside of a wildcard and allows an arbitrary number (0 or greater) of wildcards per reference object (particle, geometry, etc.). Within stardust, each wildcard is treated as opaque and is not modified in any way by the engine, besides the occasional reordering of its contiguous memory block within the wildcard array for performance reasons.

In stardust, all wildcard bytes are initialised to zero at creation! It is YOUR responsibility to initialise default values in initialisation hooks or if a contact is new, etc.

Wildcard arrays are not, and can not, be non-dimensionalised by the engine due to their opaque nature! It is YOUR responsibility to ensure your wildcard units align with the unit system you set in the solver file.

Wildcards can be declared in the solver file (with a restriction to only primitive types) or the plugin source file itself (enabling any type to be used). Within the source file wildcards are declared via the following API macro: 

    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, category)

To declare any of the categories shown above:

    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, particle)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, particle_template)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, geometry)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, triangle)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, contact)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, material)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, material_interaction)
    STARDUST_REQUIRES_WILDCARD(wildcard_name, type, simulation)


A wildcard declaration for a contact wildcard called "tangential_displacement" looks like this in the source file:

    struct ComplexType {
        float3 displacement;
        double time_in_contact;
    };
    
    STARDUST_REQUIRES_WILDCARD(tangential_displacement, ComplexType, contact)

Note that the API macros behave as regular macros in the code and therefore can be disabled via `#ifdef 0` or similar. 

If a wildcard API macro is used in your code and is unguarded, the wildcard will be declared and allocated! 

The more wildcards you declare the more memory will be used by stardust. Where possible you should try to minimise wildcard sizes and quantities!

Internally, the wildcards are addressed via an element_index and a wildcard_index. The element is the particle or contact index or similar. The wildcard_index is the index of the wildcard within that particular element's sub-array, i.e., if you declare "tangential_displacement" and then "rolling_torque", the wildcard "tangential_displacement" has a wildcard_index of 0 and "rolling_torque" has a wildcard index of 1.

As the engine may declare its own wildcards for builtin models or other purposes, the user should refrain from attempting to rely on bare indexing into the wildcard system. stardust will automatically generate the required indices at runtime. To address the wildcard of interest, such as the one from the previous example "tangential_displacement", the user should use the following tag:

    stardust_wildcards::contact::tangential_displacement

This pattern is the same for other wildcard categories, but of course, for a particle wildcard you should swap out `contact` for `particle` and so on. Note that the tag is sensitive to the case the wildcard was declared in.

Access is achieved by the appropriate API wrapper struct. For example, access to the contact wildcard we declared earlier is achieved like so:

    ComplexType wildcard = contact_data.getWildcard(stardust_wildcards::contact::tangential_displacement);

stardust returns a reference to the wildcard object. You may operate on the wildcard like you would any other memory address.

For example, when summing a wear volume per triangle in the custom contact model, you may need to allow many contacts to write to a single triangle. Attempting to sum naively would result in a race condition due to the parallel GPU execution model. In a case such as this, you can sum your quantity like so:

    const double wear_volume = ... computed earlier ...
    atomicAdd(&element_1.getWildcard(stardust_wildcards::triangle::wear_volume), wear_volume);

The user should consult the CUDA Programming Guide for further information about atomics and their use in parallel programming.

As a general rule, whenever there is a many-to-one write for a wildcard, an appropriate atomic (atomicAdd, atomicMax, atomicMin, etc.) should be employed. Note that whenever atomics are used there will be a performance penalty incurred.

### Parameters

Whilst wildcards provide zero-initialised buffers as persisted storage for the user, parameters facilitate refreshable data input into the plugin at runtime. 

Parameters should be declared similar to wildcards in the solver file or inside of the plugin itself. Within the plugin source code, a parameter can be requested with the following syntax:

    STARDUST_REQUIRES_PARAMETER(parameter_name, type, optional: category)

Note that parameter types must be primitive (refer to Plugin Datatypes). Parameters may have the following categories which are a subset of the wildcard categories:

* Particle template
* Geometry
* Material
* Material Interaction
* Global/Simulation

The category field is optional and excluding it will set the scope of the parameter as global. The category field informs the engine that this parameter must be declared per item of that category. For a global, this is once at the plugin level. For an example, to support JKR, stardust must receive an additional scalar value for the surface energy. Within the builtin file, the following code is declared:

    STARDUST_REQUIRES_PARAMETER(jkr_surface_energy, real, material_interaction)

With this declaration, the engine will throw an error if a `{ "parameters": { "jkr_surface_energy": value } }` is not included for every material interaction block in the solver file. 

To access a parameter, you should use the following syntax:

    const Type surface_energy = element_1.getParameter(stardust_parameter::jkr_surface_energy);

Parameters, unlike wildcards, are read-only on the device (from the plugin). They are useful for sending information into a plugin, however, if you need to have some property change based on some event occurring in the simulation, you are better served by copying the parameter into a wildcard at initialisation time. For example:

    const Type evolving_property = element_1.getParameter(stardust_parameters::evolving_property);
    element_1.getWildcard(stardust_wildcards::material::evolving_property) = evolving_property;

Now the wildcard can be updated over the simulation duration, tracking the evolving property.



## Performance Considerations

As with anything, you pay for what you use. All calls to the wilcard/parameter system, API wrapper structs and atomics have some associated cost which will depend on the scale of your simulation amongst other things. 

Where you are experiencing major performance degradation, you should consult the CUDA Programming Guide.

You should always try to minimise your use of wildcards and atomics. Where required, you should ensure wildcard and atomic payloads are as small as possible. 

When using custom datatypes in wildcards, the engine can not automatically guarantee alignment of memory. You should decorate your structs with `__align__(2, 4, 8, 16, etc.)` to enforce memory alignment as a hint to the compiler:

    struct __align__(16) ComplexType {
        float3 tangential_displacement;
        double time_in_contact;
    }; // compiler knows to pack this struct to achieve correct alignment

Improved memory alignment will result in better access coalesence and thus better performance in principle.



