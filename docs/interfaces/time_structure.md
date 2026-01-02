# Time and Structure Modules
These modules are responsible for two things

1. Determining what time step to take and managing the total time passage
2. Collecting input from other modules to solve the structure equations

This module is the closest thing thing which this project has to a main
component. One might think of it as the organizer of all the other modules
(side-note: I hate that when I try to use a metaphor now I need to worry that
it will sound like AI).

## Time
Time stepping is, broadly, the process of looking at the current state of a
system of time dependent equations and deciding what is the largest time step
that may be taken such that error is kept under some threshold. There are many
schemes for this. A naive approach would be to always take some fixed
time step; unfortunately for all of our sanitys, this would result in your
solver needing to take comically small time steps for the entire run such that
it would likely take well over 100 years to complete. Slightly more complex
would be some sort of defined time step growth curve, a common approach I take
when doing simple tests is to use a geometric time step growth relation where
$t^{(i+1)} = t^{(i)} + \alpha t^{(i)}$. This is certainly more effective in
some situations; however, it is still too simplistic to be useful in the case
of the stellar structure equations. 

Rather, these equations, as with most coupled non linear equations, need some
time stepping scheme which is adaptive to current system conditions. For example
there are periods of a stars life where evolution is happening much faster than
other stages (consider the speed of the pre-main sequence compared to the main
sequence). Models on the main sequence can often take time steps on the order 
of 100Myr; however, if a pre-main sequence model took such a time step it would
immediately leave the pre-main sequence, skipping over all the important physics which
takes place during that stage. 

### Time stepping Methods
The time module must decide on and adopt some time stepping approach which can
handle this. You should consider what inputs you might need in order to select
a time step. Because the equations of stellar structure can be extremely stiff
it is often required to use an implicit method rather than an explicit
method. A few possibilities

#### Variance Control
Here we try to limit the fractional change of variables from one time step to
the next. Generally this takes the form

$$
\Delta t^{(i+1)}_{\mathcal{X}} = \Delta t^{(i)}\min\left(f_{max}, \frac{\delta}{|\Delta\ln\mathcal{X}|}\right)
$$

Where $\delta$ is the maximum fractional change allowed for physical variable
from one time step to the next, $f_{max}$ is the maximum growth factor allowed
(even if all variables are stable), and $\Delta\ln\mathcal{X}$ is the magnitude
of change from the last step for some variable $\mathcal{X}$

$$
\Delta\ln\mathcal{X} = \ln\left(\frac{\mathcal{X}^{(i)}}{\mathcal{X}^{(i-1)}}\right)
$$

You would then select the most conservative timstep for all variables you want to 
limit growth on. Note that it is a key choice what variables to limit growth on.

#### Burning Limit
You might choose instead to limit your timestep such that the chemical abundnace
of any species does not change too much from one timestep to another

$$
\Delta t < \frac{\alpha X_{\text{fuel}}}{|\dot{X}_{\text{fuel}}|}
$$

#### Structural Limits
During the pre-main sequence you must limit the timestep to the thermal time
timescale of the outer layers. This is the Kelvin-helmoholtz timescale we have
or will discsuss in class.

#### Other Notes
Naivley it might be tempting to take whats called a 1st order or Godunov splitting
approach

$$
\text{Solve Structure} (t) \rightarrow \text{Nuclear Burning} (t + \Delta t) \rightarrow \text{Solve Structure} (t+\Delta t)
$$

however, the error properties of this are not ideal (scaling like
$\mathcal{O}(\Delta t)$). Generally you will find a more stable solution if you
use Strang Splitting (a 2nd order approach) such that you stagger time time
steps of microphysics solutions with structure solutions

In this approach, for each net timestep, you solve the structure equations over
two half timesteps and the microphysics equations over a single whole timestep 


### Outputs

Regardless of how you do time stepping this module must be able to, at minimum,
report the size of the current time step, the current time step number, and the
current simulation time. There may be other useful information this module
should handle which you discover throughout the term. 

Time stepping, at least in a stable way, is extremley strongly coupled to 
the solutions to the equations structure equations. One need know how how much
those solutions vary through time. Therefore this module has been coupled to 
the structure module

## Structure
The other key part of this module is the structure equations. These are the
four (five) equations of stellar structure we disscussed in class, they must
be encoded and passed to some sort of solver. I do not expect you to implement 
your own custom solver; rather, please use a solver that already exists.

There are few key parts of this

### Collecting Microphysics
The structure equations depend on loads of microphysics, some part of this module will need
to be responsible for calling the microphysics modules to get values such at the 
opacity (perhapse as a function), the specific nuclear energy generation,
and the density.

### Discritization
Numeric solutions must be solved in some discrete space, this module will need
to decide where to place the shells that compose our model. I reccomend you
work in a lagarngian scheme where you use enclosed mass as your indpendent,
radiaus-like, variable. You will need to consider how to space those shells, 
are there regions where shells can be more sparsly placed and are there
regions where they must be more densly spaced (hint: the answer is yes to 
both of those questions).

### Solution
Actual SSE codes uses something called a Henyey method to solve the structure
equations; for this course I reccomend you look into something a bit simpler
like a shooting-method. There are loads of tools which exist to solve coupled
systems of equations with a shooting method. They will not be as stable as a
Henyey method but they will be easier to implement.

### Reporting
Finally, this module must be able to report the state in some way back to the time module 
and eventually to some user so that the results can be either stepped through time
or saved and analyzed.
