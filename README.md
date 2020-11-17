# Test setup - Expert's choice
The experiment intends to create a baseline for a discretization autotuning system. The goal for the expert is to find good parameter values for h, dt, epsilon and r_c. The experts are provided with the same interface and restrictions that is used by the autotuner. They can evaluate any configuration resulting in an accuracy and runtime value with the given interface. The final value has to be within the provided search space.


## Experiment setup
- The experiment is conducted for diffusion and Gray-Scott reaction diffusion separately starting with diffusion.
- The discretization method is fixed. In this case to PSE.
- The goal is to find a configuration with an error below **10^-6** according to the L_2 norm at t=2 which is as fast as possible
- The expert may measure as many configurations as he likes
- All measurement steps are recorded, as are all other time consuming steps as visualization and analytical elaborations. The noted data are:
  - Measured **configuration** with all parameters
  - **Wall clock time** of evaluation, elaboration ...
- The final configuration is noted separatly (does not have to be the last measured one)

An [example protocol](#example-protocol) can be found below.

## Search space
The search space is limited to the same values, that are valid for the autotuning system. The expert may measure configurations outside the search space but the final configuration has to be within the limits and according to the given step sizes.

The search space is:

| parameter       | value space                    |
| --------------- | ------------------------------ |
| **1/h**         | {50, 100, 150, ..., 800}       |
| **1/dt**        | {1,2,3,4, ..., 256}            |
| **epsilon/h**   | {0.6, 0.7, 0.8, 0.9, ..., 2.0} |
| **r_c/h**       | {5, 6, 7, 8, 9}                |

this results in the following valid values for the parameters in the config file:

| parameter             | value space                        |
| --------------------- | ---------------------------------- |
| **x_particles**       | {50, 100, 150, ..., 800}           |
| **steps**             | {5000, 10000, 15000, ..., 1280000} |
| **epsilon_factor**    | {0.6, 0.7, 0.8, 0.9, ..., 2.0}     |
| **cut_radius_factor** | {5, 6, 7, 8, 9}                    |



## Setup
1. ssh falcon cluster
2. enter experiment directory:

        cd /projects/ppm/experiment/stations
3. load modules etc.

        source source_me
4. create own working directory

        cp -r T2 $(whoami)
        cd $(whoami)

5. do first test run by executing `./run` and entering the following parameters:

        ./run
        x_particles:           50
        epsilon_factor:        1
        cut_radius_factor:     5
        one div dt:            1
        discretization_method: op_pse
        creating /projects/ppm/experiment/stations/yourname/runtime_files/single_measurement/grayScott/single_measurement_50_5000_1.0_5_False_op_pse_0_1.csv
        Measuring configuration. 2.0s
        
        error L2: 6.698693125230453e-05
        time: 0.377983

6. start experiment with protocol when you are read. Don't measure any other configuration before that.

## Example protocol
The protocol should look somewhat like this. The expert may use the table below or any other form that contains the same data.
| Step | What was done?               | 1/dt   | 1/h  | epsilon/h | rc/h | wall clock time |
|------| -----------------------------|--------|------|-----------|------|-----------------|
|   -4 | Execution of configuration   | 200    |   800|        1.5|     7|        15:30 min|
|   -3 | Eamination of the previous results   ||      |           |      |         5:00 min|
|   -2 | Execution of configuration   | 56     |   600|        1.2|     8|        10:30 min|
|   -1 | Analytical elaborations (convergence plotting) |||       |      |        15:00 min|
|      |                              |        |      |           |      |                 |
|    1 |                              |        |      |           |      |                 |
|    2 |                              |        |      |           |      |                 |
|    3 |                              |        |      |           |      |                 |
|    4 |                              |        |      |           |      |                 |
|    5 |                              |        |      |           |      |                 |
|    6 |                              |        |      |           |      |                 |
|    7 |                              |        |      |           |      |                 |
|    8 |                              |        |      |           |      |                 |
|    9 |                              |        |      |           |      |                 |
|   10 |                              |        |      |           |      |                 |
|   11 |                              |        |      |           |      |                 |
|   12 |                              |        |      |           |      |                 |
|   14 |                              |        |      |           |      |                 |
|   15 |                              |        |      |           |      |                 |
|   16 |                              |        |      |           |      |                 |
|   17 |                              |        |      |           |      |                 |
|   18 |                              |        |      |           |      |                 |
|   19 |                              |        |      |           |      |                 |
|   20 |                              |        |      |           |      |                 |

## Detailed system description

### Parameter description
- **x_particles**: Particles in each dimension - so for 2D, there are `x_particles ^ 2` particles in total, placed on a regular carteesian grid.
- **epsilon_factor**: Kernel width measured as multiple of particle distance h. So 1 implies a kernel width up to the next particle. The used kernel is a gaussian kernel.
- **cut_radius_factor**: Cut-off radius/Neighborhood radius defines how many particles in the neighborhood are acutally included in the kernel evaluation. The number given is actually rc / h. Be careful here, since in contrast to some literature, it is not rc / epsilon. Also be aware that the particles are placed on a regular cartesian grid and by that only integers make sense for rc
- **one div dt**: Defines timestep size. Be careful, since you define 1/dt and not dt directly.

### PSE Kernel:
The experiment uses a gaussian kernel:

<img src="https://render.githubusercontent.com/render/math?math=\frac{1}{4 \epsilon^2 \pi}e^{-\frac{distance^2}{4\epsilon^2}}">

### Timestepping
Timestepping is done using explicit euler

### Accuracy calculation
The accuracy is calculated against a reference configuration. For the calculation, the L2 norm over a 50x50 grid (which is common between all valid values for h) is used. As accuracy reference, the following configuration is used:

    x_particles:           1600
    epsilon_factor:        0.8
    cut_radius_factor:     9
    one div dt:            1024


### Gray Scott simulation

    // Diffusion constants
    Du = 4 * 1e-5;
    Dv = 2 * 1e-5;

    // Physical constants
    K = 0.055;
    F = 0.03;

Initial condition:

[Plot](square_circle.pdf)

<img src="https://render.githubusercontent.com/render/math?math=\frac{1}{0.16 \cdot|\pi|} e^{-\frac{distance^2}{0.16}}">

### Diffusion simulation

    // Diffusion constant
    constexpr double diff_D = 1e-4;

Initial condition:

[Plot](gaussian.pdf)

<img src="https://render.githubusercontent.com/render/math?math=q(x, y) = \frac{1}{\frac{x^4+y^4}{0.15^4}%2B1}">
