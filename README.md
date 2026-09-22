# Stokes

A real-time fluid simulation. The incompressible Navier-Stokes equations solved
on the GPU — advection, vorticity confinement, and an iterative pressure
projection — every frame.

Live: https://bxzex.github.io/stokes/

## The step

Each frame runs a chain of full-screen shader passes over floating point
textures, splitting the equations the way Stable Fluids does:

1. **Curl and vorticity confinement.** Numerical diffusion eats the small
   eddies, so energy is pushed back along the gradient of the curl to keep the
   detail alive.
2. **Divergence** of the velocity field.
3. **Pressure**, by Jacobi sweeps on the Poisson equation.
4. **Gradient subtraction**, which removes the divergence and leaves the field
   incompressible.
5. **Semi-Lagrangian advection** of velocity and then of dye — look back along
   the velocity and fetch what was there, which is unconditionally stable at any
   time step.

Velocity is `RG16F`, dye `RGBA16F`, pressure and divergence `R16F`, each
double-buffered and ping-ponged. Walls are free-slip: the component normal to
the boundary is reflected.

## The discretisation, and getting it wrong first

The projection only works if the divergence operator, the Laplacian the Jacobi
sweep solves, and the gradient operator are consistent with each other. I first
used central differences for divergence and gradient against a compact five
point Laplacian, which is what most browser fluid demos do. Measuring it showed
why that is a mistake: the mean absolute divergence fell by 24% and then
**started rising again** with more sweeps, because pressure on odd and even
cells is decoupled under that pairing and the extra iterations sharpen the wrong
solution.

Switching to a forward difference for divergence and a backward difference for
the gradient composes to exactly the five point Laplacian. The projection then
behaves the way it should:

| Jacobi sweeps | divergence removed |
|---|---|
| 30 | 13.4% |
| 80 | 28.5% |
| 200 | 36.1% |
| 400 | 39.8% |

Monotone — more sweeps always help. It plateaus rather than reaching zero
because Jacobi converges slowly on a grid this size and the fields are half
precision, which is the honest trade for running at frame rate.

Pressure is also clamped: under strong forcing and many sweeps it would run past
the half float range and poison the velocity field with NaN. That one showed up
as the screen going black in testing.

## Notes

One HTML file. No libraries, no build step. Needs WebGL2 and
`EXT_color_buffer_float`, and says so plainly if they are missing. Drag to push
the fluid; it drifts on its own until you do.

Built by [bxzex](https://bxzex.com).
