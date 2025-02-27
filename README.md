# DataDrivenDiffEq.jl

[![Build Status](https://github.com/SciML/DataDrivenDiffEq.jl/workflows/CI/badge.svg)](https://github.com/SciML/DataDrivenDiffEq.jl/actions?query=workflow%3ACI)
[![codecov.io](http://codecov.io/github/SciML/DataDrivenDiffEq.jl/coverage.svg?branch=master)](http://codecov.io/github/JuliaDiffEq/DataDrivenDiffEq.jl?branch=master)
[![DOI](https://zenodo.org/badge/212827023.svg)](https://zenodo.org/badge/latestdoi/212827023)
[![ColPrac: Contributor's Guide on Collaborative Practices for Community Packages](https://img.shields.io/badge/ColPrac-Contributor's%20Guide-blueviolet)](https://github.com/SciML/ColPrac)


DataDrivenDiffEq.jl is a package in the SciML ecosystem for data-driven differential equation
structural estimation and identification. These tools include automatically discovering equations
from data and using this to simulate perturbed dynamics.

For information on using the package,
[see the stable documentation](https://datadriven.sciml.ai/stable/). Use the
[in-development documentation](https://datadriven.sciml.ai/dev/) for the version of
the documentation which contains the un-released features.

## Quick Demonstration

```julia
## Generate some data by solving a differential equation
########################################################
using DataDrivenDiffEq
using ModelingToolkit
using OrdinaryDiffEq

using LinearAlgebra

# Create a test problem
function lorenz(u,p,t)
    x, y, z = u

    ẋ = 10.0*(y - x)
    ẏ = x*(28.0-z) - y
    ż = x*y - (8/3)*z
    return [ẋ, ẏ, ż]
end

u0 = [1.0;0.0;0.0]
tspan = (0.0,100.0)
dt = 0.1
prob = ODEProblem(lorenz,u0,tspan)
sol = solve(prob, Tsit5(), saveat = dt, progress = true)


## Start the automatic discovery
ddprob = ContinuousDataDrivenProblem(sol)

@variables t x(t) y(t) z(t)
u = [x;y;z]
basis = Basis(polynomial_basis(u, 5), u, iv = t)
opt = STLSQ(exp10.(-5:0.1:-1))
ddsol = solve(ddprob, basis, opt, normalize = true)
print(ddsol, Val{true})
```

```
Explicit Result
Solution with 3 equations and 7 parameters.
Returncode: success
Sparsity: 7.0
L2 Norm Error: 26.7343984476783
AICC: 1.0013570199499398

Model ##Basis#366 with 3 equations
States : x(t) y(t) z(t)
Parameters : 7
Independent variable: t
Equations
Differential(t)(x(t)) = p₁*x(t) + p₂*y(t)
Differential(t)(y(t)) = p₃*x(t) + p₄*y(t) + p₅*x(t)*z(t)
Differential(t)(z(t)) = p₇*z(t) + p₆*x(t)*y(t)

Parameters:
   p₁ : -10.0
   p₂ : 10.0
   p₃ : 28.0
   p₄ : -1.0
   p₅ : -1.0
   p₆ : 1.0
   p₇ : -2.7
```
