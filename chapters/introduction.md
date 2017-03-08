# Introduction

GPUE is a Gross-Pitaevskii Equation (GPE) solver that uses General-Purpose Graphical Processing Units (GPU) architecture (the name is an amalgam of the two). The GPE is a non-linear Schroedinger equation term used in studying BEC systems:

$$
i\hbar \frac{\partial \Psi(r,t)}{\partial t} = \left( -\frac{\hbar}{2m}\nabla ^2 + V(r) + g|\Psi(r,t)|^2 \right) \Psi(r,t)
$$

Here, $$\Psi(r,t)$$ is the wavefunction, $$m$$ is the mass and $$g = \frac{4\pi\hbar^2a_s}{m}$$ is a nonlinear interaction term. With $$g=0$$, the GPE becomes the linear Scr\"odinger equation.

In the case of GPUE, the equation is solved by the split-step operator method, which is a psuedo-spectral method used to solve partial differential equations.
