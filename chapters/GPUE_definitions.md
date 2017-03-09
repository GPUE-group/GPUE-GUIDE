# GPUE Definitions

GPUE assumes an $$^{87}$$Rb condensate, whose values may be found in `src/init.cu`. The relevant values are: 

* Mass: ($$m =1.4431607e^{-25}$$)
* Scattering Length: ($$a_s = 4.76e^{-9}$$)
* Coupling constant: ($$g = 4.52235e^{-44}$$)

All other values can be found in `data/Params.dat`. It should be mentioned that these values are dependent on how you run GPUE, as defined in the previous section.

## GPUE Classes
GPUE has 4 primary classes found in `include/ds.h` and `src/ds.cc`:

* Grid (`par`) -- This is where we keep all auxiliary variables in one large unordered map. These values are mostly set in `src/parser.cc` and `src/init.cu`
* Wave (`wave`) -- This is where the wavefunction data both on the cpu and gpu
* Cuda (`cupar`) -- This is where we keep CUDA-specific variables like `grid` and `threads`
* Op (`opr`) -- This is where all operators are held in an unordered map of function pointers, to be described below

## Operator Definitions
All operators can be found in `src/operators.cc` and are called from `src\init.cu` during the initialization loops in `init_2d(...)` and `init_3d(...)`. The operators are all functions with the following parameter list:
```
double operator(Grid &par, Op &opr, int i, int j, int k)
```
This is so we may store them in unordered maps of function pointers in `include/ds.h` and call them in the initialization loops like so:
```
pAy[index] = opr.pAy_fn("rotation")(par, opr, i, j, k);
```
Which allows us to store multiple distributions of our variables to be read in at runtime thus avoiding unnecessary building of GPUE. Even though these are generated on the CPU, they are immediately transmitted to the GPU for processing. 

For the most part, these operators are initialized into the unordered map with the `set_fns()` function found in `src/ds.cc`, and called in `src/init.cu` at the start of the `init_2d(...)` and `init_3d(...)` functions.

## Procedure for adding new operators

Each operator is set in the `src/operators.cc` file. In essence, the procedure for creating a new field is the following:

1. Create the function for the operator by calling the `double *` values of `x`, `y`, and `z` (if 3d) and any other necessary parameter found in `Grid &par`. Return a single `double` value that corresponds to the field at the location of `x[i]`, `y[j]`, and `z[k]`

2. Update `include/operators.h` with the new function

3. Update the `set_fns()` function in `src/ds.cc` to add a new string that calls the operator to the maps for `Op_pAy_fns`, `Op_pAx_fns`, `Op_pAz_fns`, `Op_Ay_fns`, `Op_Ax_fns`, and `Op_Az_fns`. This string will be used with the `-A` flag when setting distributions at run-time.

There are more efficient ways to do this initialization; however, this is the version that is currently in the code. If an intrepid intern or student wishes to tackle this problem, they should go right ahead and update this guide as necessary.

## Procedure for adding new potentials

The procedure for adding new potentials is much the same as that for adding new operators, with the exception that a new wavefunction definition must be set in the `set_fns()` function for the `Wave` class. If this step is neglected, it will take a much longer time to converge to the ground state in imaginary time evolution!

The potential is still set in the `Op::set_fns()` function like the operators, but will be modified with the `-v` flag. As mentioned before, this part of the code may need some elbow-grease. We'll get there.

## Specific Operators

### Rotation

The Rotational gauge field is the "standard" case (as defined by the helpfile and `src/parser.cc`) for GPUE and will be assumed unless otherwise changed with the `-A` or `-v` flag.

#### 2D
#### 3D
For the standard rotational case in 3d, we assume a spherical condensate with a harmonic trap.

### Torus

To create a torus trapping potential, we wrap a 2d harmonic trap in a circular structure with the following code:

```
    double rad = sqrt((x[i] - xOffset) * (x[i] - xOffset)
                      + (y[j] - yOffset) * (y[j] - yOffset)) - 0.5*rMax*fudge;
    double omegaR = sqrt(omegaX*omegaX + omegaY*omegaY);
    double V_r = omegaR*rad;
    V_r = V_r*V_r;

    double V_z = omegaR*(z[k]+zOffset);
    V_z = V_z * V_z;
    return 1 * 0.5 * mass * ( V_r + V_z) +
           0.5 * mass * (pow(opr.Ax_fn(par.Afn)(par, opr, i, j, k),2) +
                         pow(opr.Az_fn(par.Afn)(par, opr, i, j, k),2) +
                         pow(opr.Ay_fn(par.Afn)(par, opr, i, j, k),2));

```

To match this, the torus wfc is updated here:

```
    // We will now create a 2d projection and extend it in a torus shape
    double rad = sqrt((x[i] - xOffset) * (x[i] - xOffset)
                      + (y[j] - yOffset) * (y[j] - yOffset)) - 0.5*rMax*fudge;

    wfc.x = exp(-( pow((rad)/(Rxy*a0x*0.5),2) +
                   pow((z[k])/(Rxy*a0z*0.5),2) ) );
    wfc.y = -exp(-( pow((rad)/(Rxy*a0x*0.5),2) +
                    pow((z[k])/(Rxy*a0z*0.5),2) ) );

```

Here, we have used the `-F` "fudge factor" to modify the internal radius of the torus for testing purposes. This should *probably* be removed in the future.

In principle, the component of the GPE that will be dealt with in real space, notated as $$V$$ is the following:

$$
V = \frac{m}{2}\left(\omega_r^2 r^2 + A^2\right)
$$

The initial wavefunction is defined as:

$$
\Psi = e^{-\left(\frac{2r}{R_{xy} a_{0x}} + \frac{2z}{R_{xy} a_{0x}}\right)} - ie^{-\left(\frac{2r}{R_{xy} a_{0x}} + \frac{2z}{R_{xy} a_{0x}}\right)}
$$

were $$R_{xy}$$ is a condensate scaling factor and $$a_{0x}$$ is the harmonic oscillator length in the $$x$$-direction. Both of these parameters are defined in `src/init.cu`.
