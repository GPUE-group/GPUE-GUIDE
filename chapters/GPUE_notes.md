# Notes on Running GPUE

GPUE uses the Compute Unified Device Architecture (CUDA) software platform built with the `nvcc` compiler. For this reason, GPUE must be run on a platform with an appropriate NVIDIA graphics processor

## Building GPUE
In principle, GPUE can be built with a simple `make` command; however, GPUE requires access to a CUDA installation, so the following line must be updated to match the location of your current installation

```
CUDA_HOME = /work/scratch/user/cuda_8.0/
```

The binary will be generated in the same directory and may be run as indicated in the following section

## Running GPUE

Like many commandline programs, GPUE has a host of flags that may be turned on and off. The default settings may be found with `./gpue -h`, or ``help,'' with the full output here (It looks better in-terminal): 


```
                                      GPUE
          Graphics-Processing Unit Gross--Piteavskii Equation solver
Options:

-A rotation     Set gauge field mode [beta]
-a              Set flag to graph [deprecated]
-b 2.5e-5       Set box size (xyzMax in 3d)
-C 0            Set device (Card) for GPU computing
-c              Set coords (1, 2, 3 dimensions)
-D 0.0          Set's offset for kill (-K flag) distance radially
-d data         Set data directory / where to store data (here in data/)
-e 1            Set esteps, number of real-time evolution steps
-f              Unset write to file flag
-G 1            Set GammaY, ratio of omega_y to omega_x
-g 1            Set gsteps, number of imaginary-time evolution steps
-h              Print help menu, exit GPUE
-H phrase       Print help menu, searching for "phrase"
-i 1            Set interaction strength between particles
-K              Selects vortex with specified ID to be killed
-k 0            Set kick_it, kicking for Moire lattice simulations
                0 = off, 1 = periodic kicking, 2 = single kick
-L 0            Set l, vortex winding [2*pi*L]
-l              Set ang_mom flag to use angular momentum
-m              Set 2d_mask for vortex tracking
-n 1            Set N, number of particles in simulation
-O 0            Set angle_sweep, kicking potential rotation angle
-P 0            Set laser_power, strength of kicking [hbar * omega_perp]
-p 100          Set printSteps, frequency of printing
-Q 0            Set z0_shift, z shift of kicking potential (bad flag, I know)
-R              Set ramping flag for imaginary time evolution
-r              Set read_wfc to read wavefunction from file
-S 0            Set sepMinEpsilon, kicking potential lattice spacing scaling
-s              Set gpe, flag for using Gross-Pitaevskii Equation
-T 1e-4         Set gdt, timestep for imaginary-time
-t 1e-4         Set dt, timestep for real-time
-U 0            Set x0_shift, x shift of kicking potential
-u              Performs all unit tests
-V 0            Set y0_shift, y shift of kicking potential
-v              Set potential [beta]
-W              Set write_it, flag to write to file
-w 0            Set Omega_z, rotation rate reltive to omega_perp
-X 6.283        Set omega_x
-x 256          Set xDim, dimension in X (power of 2, please!)
-Y 6.283        Set omega_y
-y 256          Set yDim, dimension in Y (power of 2, please!)
-Z 6.283        Set omega_z, confinement in z-dimension
-z 256          Set zDim, dimension in Z (power of 2, please!)


Notes:

- Parameters specified above represent the default values for the classic 
  linear Schrodinger equation and may need to be modified to match your
  specific problem.

- We use real units with an Rb87 condensate

- You may generate a simple vortex lattice in 2d with the following command:
  ./gpue -x 512 -y 512 -g 50000 -e 1 -p 5000 -W -w 0.5 -n 1e5 -s -l -Z 10

- Thanks for using GPUE!

```

As indicated in the helpfile, gpue may be run by concatenating many flags together. The following gauge fields are allowed:

* `rotation` -- This provides a field identical to a rotating BEC
* `file` -- This reads in the gauge field from `src/Axgauge`, `src/Aygauge`, and `src/Azgauge`
* `dynamic` -- This will parse the equation strings found in `src/Axgauge`, `src/Aygauge`, and `src/Azgauge`

When setting a potential with the `-v` flag, an appropriate trial wavefunction is generated for the trap. Currently, the following potentials are allowed:

* `harmonic` -- This is the standard $$\frac{m}{2}\left((\omega_x x)^2 + (\omega_y y)^2 + (\omega_z z)^2\right)$$
* `torus` -- This extends the harmonic trap in a torus as discussed below


## Running GPUE on a cluster
If running GPUE on a supercomputing cluster, there are a number of job schedulers available. Assuming that you are using SLURM, the following script may be used:

```
\\TODO
```

