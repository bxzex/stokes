# Stokes

A real-time fluid simulation on the GPU. Drag across it to push the dye around.

https://bxzex.github.io/stokes/

It follows Stable Fluids: vorticity confinement, divergence, Jacobi pressure solves, gradient subtraction, then semi-Lagrangian advection, all as shader passes over half-float textures.

My first version used central differences like most browser fluid demos do, and more pressure iterations actually made the divergence worse. Switching to forward and backward differences that match the Laplacian fixed it, and now more iterations always help. I also clamp pressure, because it used to overflow half precision and turn the screen black.

Needs WebGL2 with float render targets.
