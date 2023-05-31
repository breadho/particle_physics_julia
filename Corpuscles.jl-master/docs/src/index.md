![](https://github.com/JuliaPhysics/Corpuscles.jl/raw/master/docs/src/assets/corpuscles.png)

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://JuliaPhysics.github.io/Corpuscles.jl/stable)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://JuliaPhysics.github.io/Corpuscles.jl/dev)
[![Build Status](https://github.com/JuliaPhysics/Corpuscles.jl/workflows/CI/badge.svg)](https://github.com/JuliaPhysics/Corpuscles.jl/actions)
[![Codecov](https://codecov.io/gh/JuliaPhysics/Corpuscles.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaPhysics/Corpuscles.jl)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3933364.svg)](https://doi.org/10.5281/zenodo.3933364)

# Corpuscles.jl

**Corpuscles.jl** is a package which gives easy access to particle
properties and identification codes summarised and defined by the
[Particle Data Group (PDG)](https://pdg.lbl.gov) collaboration.
The cleaned CSV versions of these data are provided by courtesy
of the [Scikit-HEP project](https://scikit-hep.org) and are part
of the [Particle](https://github.com/scikit-hep/particle) Python
module which inspired us to create a similar package for the
[Julia Language](https://www.julialang.org). **Corpuscles.jl**
is by far not as feature complete as **Particle**, but we add
functionality continuously, as needed. Feel free to create an
[Issue](https://github.com/JuliaPhysics/Corpuscles.jl/issues/new) or pull request if
you find any bugs or have suggestions to improve.


## Usage

The `Particle` struct can be used to create a particle. If an integer value is
passed, it will be interpreted as [PDG ID](http://pdg.lbl.gov/2019/reviews/rpp2019-rev-monte-carlo-numbering.pdf), which is the primary particle
encoding in **Corpuscles.jl**:

```julia
julia> using Corpuscles

julia> p = Particle(12)
Particle(12) 'nu(e)'
```

To get an overview of the available particle information, use `print()`:

```julia
julia> print(p)
Name:    K(4)*(2045)
PDG ID:  -319
LaTeX:   $\bar{K}_{4}^{*}(2045)^{0}$
Status:  Common
Width = 198.0 MeV ± 30.0 MeV
Q (charge) = 0//1 e
Composition = Ds
Isospin = 1//2
Mass = 2045.0 MeV ± 9.0 MeV
P (space parity) = 1
```

The properties are accessible via attributes:

```julia
julia> fieldnames(Particle)
(:pdgid, :mass, :width, :charge, :isospin, :parity, :gparity, :cparity, :antiprop, :rank, :status, :name, :quarks, :latex)

julia> p.quarks
"Ds"

julia> p.isospin
1//2

julia> p.mass
2045.0 MeV ± 9.0 MeV
```

There are tons of helper functions to check other properties:

```julia
julia> filter(hasstrange, particles())
257-element Array{Particle,1}:
 Particle(3224) Sigma(1385)
 Particle(23124) Lambda(1890)
 Particle(-13324) Xi(1820)
 Particle(-329) K(4)*(2045)
 ⋮
 Particle(13124) Lambda(1690)
 Particle(-100321) K(1460)
 Particle(20433) D(s1)(2460)
julia> filter(islepton, particles())
16-element Array{Particle,1}:
 Particle(18) nu(tau')
 Particle(15) tau
 Particle(11) e
 Particle(13) mu
 Particle(14) nu(mu)
 Particle(17) tau'
 ⋮
 Particle(12) nu(e)
 Particle(-14) nu(mu)
 Particle(-16) nu(tau)
 Particle(-12) nu(e)
 Particle(-13) mu
```

## Units

For some properties like `mass` and `width` we use the
[Unitful](https://github.com/PainterQubits/Unitful.jl) package, which makes it
easy to combine values with physical units:

```julia
julia> typeof(p.mass)
Corpuscles.MeasuredValue{𝐋^2 𝐌 𝐓^-2}

julia> p.mass
2045.0 MeV ± 9.0 MeV

julia> p.mass.value
2045.0 MeV

julia> p.mass.lower_limit
9.0 MeV

julia> p.mass.upper_limit
9.0 MeV
```

and also `Base.isless` and `Base.isapprox` are implemented so that the
lower and upper limits are taken into account, as seen here:

```julia
julia> using Unitful

julia> p.mass
2045.0 MeV ± 9.0 MeV

julia> p.mass > 2036u"MeV"
false

julia> p.mass > 2035u"MeV"
true
```

## Particle Codes

Corpuscles currently supports conversions of Pythia and Geant3 codes to PDG
using the `Pythia` and `Geant3ID` types. The corresponding number is simply
passed to the type and then to the `Particle` itself. The PDG ID will be
converted if possible, otherwise an error will be raised.

```julia
julia> Particle(Geant3ID(5))
Particle(-13) 'mu'
```

## Finding Particles

The `particles()` functions returns a `Vector` containing all the particles
of the currently selected catalog. To search for particles, the `filter()`
function comes in hand which can be combined with string comparison functions
like `startswith()` or `occursin()`.

Here is an example how to find all particles with names starting with `"nu"`:

```julia
julia> filter(p->startswith(p.name, "nu"), particles())
6-element Array{Particle,1}:
 Particle(-14) 'nu(mu)'
 Particle(-16) 'nu(tau)'
 Particle(-12) 'nu(e)'
 Particle(14) 'nu(mu)'
 Particle(16) 'nu(tau)'
 Particle(12) 'nu(e)'
```

A more powerful way to filter particles based
on patterns in their name is using [regular
expressions](https://docs.julialang.org/en/v1/manual/strings/#Regular-Expressio ns-1)
with e.g. `occursin()`:

```julia
julia> filter(p->occursin(r"D\(\d*\)", p.name), particles())
10-element Array{Particle,1}:
 Particle(-10421) 'D(0)*(2300)'
 Particle(-10411) 'D(0)*(2300)'
 Particle(425) 'D(2)*(2460)'
 Particle(10411) 'D(0)*(2300)'
 Particle(10421) 'D(0)*(2300)'
 Particle(10423) 'D(1)(2420)'
 Particle(-425) 'D(2)*(2460)'
 Particle(-10423) 'D(1)(2420)'
 Particle(415) 'D(2)*(2460)'
 Particle(-415) 'D(2)*(2460)'
```

Another useful feature is the `Particle(n::String, anti=false)` which directly
creates a (anti-)particle from a given name -- as long as it exists in the
currently loaded particle dataset:

```julia
julia> Particle("mu") == Particle(13)
true

julia> -Particle("mu") == Particle(-13)
true

julia> Particle("anti-mu") == Particle(-13)
true

julia> Particle("mu~") == Particle(-13)
true

julia> Particle("muon") == Particle(13)
true

julia> Particle("mu-") == Particle(13)
true

julia> Particle("mu+") == -Particle(13)
true
```
