# GPUE definitions

GPUE assumes an $$^{87}$$Rb condensate, whose values may be found in `src/init.cu`. The relevant values are: 

* Mass: ($$m =1.4431607e^{-25}$$)
* Scattering Length: ($$a_s = 4.76e^{-9}$$)
* Coupling constant: ($$g = 4.52235e^{-44}$$)

All other values can be found in `data/Params.dat`. It should be mentioned that these values are dependent on how you run GPUE, as defined in the previous section.

\subsection{GPUE classes}
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
Which allows us to store multiple distributions of our variables to be read in at runtime thus avoiding unnecessary building of GPUE.
