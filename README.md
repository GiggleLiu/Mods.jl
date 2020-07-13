# Mods

## Important Notice: Breaking changes coming!

I plan to reimplement this module so that a mod-N number is represented
as `Mod{N}(x)` instead of `Mod(N,x)`. In this way `Mod{N}` becomes a type.

I will release this new implementation with version numbers of the form 1.x.x
to signify the break with the previous implementation (versions 0.x.x).



[![Build Status](https://travis-ci.org/scheinerman/Mods.jl.svg?branch=master)](https://travis-ci.org/scheinerman/Mods.jl)

[![Coverage Status](https://coveralls.io/repos/scheinerman/Mods.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/scheinerman/Mods.jl?branch=master)

[![codecov.io](http://codecov.io/github/scheinerman/Mods.jl/coverage.svg?branch=master)](http://codecov.io/github/scheinerman/Mods.jl?branch=master)


Easy modular arithmetic for Julia.

Construct an immutable `Mod` object with `Mod(val,mod)`.  Both `val`
and `mod` must `Integer` values.
```julia
julia> using Mods

julia> Mod(4,23)
Mod(4,23)

julia> Mod(-1,23)
Mod(22,23)

julia> Mod(99,12)
Mod(3,12)

julia> x = Mod(4,10)
Mod(4,10)

julia> x.val
4

julia> x.mod
10
```

With just a single argument, `Mod` creates a zero element of the given
modulus.
```julia
julia> Mod(17)
Mod(0,17)
```


## Operations

### The basic four

`Mod` objects can be added, subtracted, mulitplied, and divided with
one another. The two `Mod` operands must have the same modulus.
```julia
julia> x = Mod(8,10); y = Mod(6,10);

julia> x+y
Mod(4,10)

julia> x-y
(2,10)

julia> x*y
Mod(8,10)

julia> Mod(5,10) + Mod(5,11)
ERROR: Cannot operate on two Mod objects with different moduli
```

Division can result in an error if the divisor is not invertible. A
`Mod` object `x` can be checked for invertibility using
`is_invertible(x)`. To find the inverse of `x` (assuming it exists)
use `inv(x)`. **WARNING**: The syntax `x'` is no longer valid for finding
the inverse of `x`.
```julia
julia> x = Mod(8,10); y = Mod(6,10);

julia> x/y
ERROR: Mod(6,10) is not invertible

julia> x = Mod(8,10); y = Mod(3,10);

julia> x/y
Mod(6,10)

julia> inv(y)
Mod(7,10)
```

We also support unary minus.
```julia
julia> x = Mod(3,10);

julia> -x
Mod(7,10)
```

### Mixed Integer/Mod arithmetic

The basic four operations may also be performed between a `Mod` object
and an `Integer`. The calculation proceeds as if the `Integer` has the
same modulus as the `Mod` object.
```julia
julia> x = Mod(3,10);

julia> x+9
Mod(2,10)

julia> 4x
Mod(2,10)

julia> 3-x
Mod(0,10)

julia> x/7
Mod(9,10)
```




### Exponentiation

Use `x^k` to raise a `Mod` object `x` to an `Integer` power `k`. If
`k` is zero, this always returns `Mod(1,m)` where `m` is the modulus
of `x`. Negative exponentiation succeeds if and only if `x` is
invertible.
```julia
julia> x = Mod(3,100)
Mod(3,100)

julia> x^10
Mod(49,100)

julia> x^-2
Mod(89,100)

julia> x = Mod(5,100)
Mod(5,100)

julia> x^-3
ERROR: Mod(5,100) is not invertible

julia> Mod(0,10)^0
Mod(1,10)
```

### Equality and hashing

Two `Mod` objects can be compared for equality with either `==` or
`isequal`.
```julia
julia> Mod(3,10) == Mod(3,11);
false

julia> Mod(3,10) == Mod(-7,10)
true
```

We can also compare `Mod` objects with `Integer` objects:
```julia
julia> Mod(3,10) == -7
true

julia> Mod(3,10) == 7
false
```


We also define `hash` for `Mod` objects so they can be stored in sets
and used as keys in a dictionary.
```julia
julia> A = Set{Mod}()
Set{Mod}({})

julia> push!(A, Mod(3,10))
Set{Mod}({Mod(3,10)})
```


### Chinese Remainder Theorem calculations

The Chinese Remainder Theorem gives a solution to the following
problem. Given integers `a, b, m, n` with `gcd(m,n)==1` find an
integer `x` such that `mod(x,m)==mod(a,m)` and
`mod(x,n)==mod(b,n)`. We provide the `CRT` function to solve this
problem as illustrated here with `a=3`, `m=10`, `b=5`, and `n=17`:

```julia
julia> s = Mod(3,10); t = Mod(5,17);

julia> CRT(s,t)
Mod(73,170)
```

We find that `mod(73,10)` equals `3` and `mod(73,17)` equals `5` as
required. The answer is reported as `Mod(73,170)` because any value of
`x` congruent to 73 modulo 170 is a solution.

The `CRT` function can be applied to any number of arguments so long
as their moduli are pairwise relatively prime. If called with no
arguments, `CRT` returns `Mod(0,1)` since all integers are congruent
to 0 modulo 1.
