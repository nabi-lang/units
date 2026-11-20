# units

[![CI](https://github.com/nabi-lang/units/actions/workflows/ci.yml/badge.svg)](https://github.com/nabi-lang/units/actions/workflows/ci.yml) [![packages.nabi-lang.org](https://img.shields.io/github/v/tag/nabi-lang/units?label=packages.nabi-lang.org&sort=semver)](https://packages.nabi-lang.org/packages/units) [![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue)](#license)

Physical quantities with units for Nabi. A `Quantity` carries its dimension,
the exponents of the seven SI base quantities, in its type: a formula that adds
a length to a time, or passes an energy where a force goes, does not compile,
and the check costs nothing when the program runs. Units read at run time
(`"km/h"`, `"kg*m/s^2"`) go through `DynUnit` and `Measure`, which check the
dimension when they run and throw a `UnitError` that names both sides and the
factor between them.

## Add it

```sh
nabi add units
```

Needs Nabi 0.1 or later. Pure Nabi: every target works, wasm included.

## Usage

```nabi
use units::kinds::{Energy, Length, Mass, Time, Velocity};
use units::symbols::{h, km, mi, kg, m, s²};

fn speed(d: Length, t: Time) -> Velocity { d / t }
fn kinetic(mass: Mass, v: Velocity) -> Energy { 0.5 * mass * v.squared() }

let v = speed(42.195km, 2h);
println(v);                    // 5.860416666666667 m/s
println(v → km / h);           // 21.0975 km/h
println(5km → mi);             // 3.1068559611866697 mi
println(kinetic(2kg, 3m / 1s)); // 9 J
let newton = kg * m / s²;      // a static unit; prints `kg·m/s²`
```

Number suffixes (`3km`, `9.81m / 1s / 1s`, `5kΩ`, `2μF`, `20degC`) give
quantities; `units::symbols` has the same units as constants for `.to(..)`,
`.value_in(..)` and unit algebra. Values are held in coherent SI units, and a
quantity prints with its SI unit, simplified to a named one where that is
shorter (`N`, `J/K`, `W/(m·K)`).

```nabi
let p = 2kN / 4cm²;            // 5000000 Pa
// let mistake = 3m + 2s;      // does not compile: `+` needs one dimension
```

Temperatures on scales with an offset are readings, not amounts: two readings
subtract to an interval in kelvin, an interval adds to a reading, and two
readings do not add.

```nabi
use units::symbols::degF;

let t = 20degC;
println(t → degF);             // 68 °F
println(30degC - 20degC);      // 10 K
println(t + 5K);               // 25 °C
// 20degC + 10degC              does not compile
```

Units known only at run time:

```nabi
use units::{Measure, UnitError};

let g = try Measure::parse("9.81 m/s^2");
println(try g.to_str("ft/s^2"));   // 32.18503937007874 ft/s²
try g.to_str("s") catch e: UnitError {
    println(e); // cannot convert m/s² (acceleration, L·T⁻²) to s (time, T): the dimensions differ; divide by a jerk (m/s³) first
};
```

CODATA constants are quantities too, with their uncertainties as data:

```nabi
use units::constants::{c, h, k_B, lookup};

println(h * c / 500nm);            // photon energy, in J
println(k_B * 25degC.absolute());  // thermal energy at room temperature
println((lookup("G") ?? panic("G")).uncertainty); // 0.0000000000000015
```

## What is in it

| Item | Contents |
|---|---|
| `Quantity<V, L, M, T, I, K, N, J>` | a value of type `V` in SI units; the seven exponents are length, mass, time, current, temperature, amount, luminous intensity. `* / + -`, comparisons, `√`, `.squared()`, `.powi::<n>()`, `.to(u)`, `.value_in(u)`, `f"{q:.3}"`, `f"{q:.2e}"`, `.format(spec)` for a spec chosen at run time, `.measure()` |
| `kinds` | names for 46 dimensions: `Length`, `Velocity`, `Force`, `Pressure`, `Entropy`, ...; each takes the value type, `Length<f32>`, `f64` when left out |
| `symbols` | about 170 unit constants: `m`, `km`, `μm`, `inch`, `mi`, `kg`, `lb`, `s`, `ms`, `h`, `Hz`, `N`, `J`, `kWh`, `eV`, `W`, `hp`, `Pa`, `bar`, `atm`, `psi`, `V`, `Ω`, `F`, `T`, `K`, `mol`, `deg`, `kn`, `mph`, `m²`, `s²`, `s⁻¹`, ...; `degC`, `degF`, `degR`, `kelvin` scales |
| `Unit<..>` | a static unit: symbol and scale; `*` and `/` combine units, `.of(x)`, `.array(a)` |
| `DynUnit`, `Measure` | units and values with a run-time dimension: `DynUnit::parse`, `Measure::parse`, `.to`, `.to_str`, `.fixed()` |
| `Dim` | a dimension as a value: algebra, `.symbol()`, `.ascii()`, `.name()` |
| `Temperature`, `temperature::Scale` | readings on the Kelvin, Celsius, Fahrenheit and Rankine scales |
| `constants` | `c`, `h`, `hbar`, `e`, `k_B`, `N_A`, `R`, `F`, `sigma`, `G`, `alpha`, `mu_0`, `epsilon_0`, `m_e`, `m_p`, `m_n`, `m_u`, `a_0`, `R_inf`, ...; `codata()`, `lookup` |
| `unit::catalogue`, `unit::prefix` | the unit catalogue (NIST SP 811 factors) and the 24 SI prefixes, quecto to quetta |
| `literals` | number suffixes (feature `literals`, on by default) |
| `ops` | `q → u` conversion and `x°` degrees (feature `ops`, on by default) |
| `UnitError` | what went wrong: `Mismatch`, `Unknown` (with a suggestion), `Syntax`, `Value` |

### Reading a type error

A static mismatch names the kinds, `expects Velocity, found Length`. A
dimension with no name in `kinds` shows its exponents in order `L, M, T, I, K,
N, J`: `Quantity<f64, 1, 0, 1, 0, 0, 0, 0>` is a length times a time (m·s).

### Operators

| Operator | Meaning | Level |
|---|---|---|
| `q → u` (U+2192) | `q.to(u)`: the quantity in unit `u`; a `Measure` on the right gives a `Result` | `\|`: `a + b → km` converts the sum |
| `x°` (U+00B0) | `x` degrees as an angle in radians, a dimensionless quantity | postfix `?`: `-30°` is `-(30°)` |
| `√q` | square root; every exponent must be even, else it does not compile | core's prefix `√` |
| `q ** n` | a `Measure`: `n` is a value, so the dimension is too. `q.powi::<n>()` and `q.squared()` stay static | `**` |

### Literals and other packages

A suffix is a name, so `m²`, `μs`, `kΩ`, `2bar` and `1oz` work and `%` or
`°C` do not (`percent`, `degC`). No suffix may start with `e` (`2e3` is an
exponent): the electronvolt has `keV`, `MeV`, `GeV`, `TeV` but no `eV` suffix
(`eV.of(2.0)`).

The `signal` package registers `hz`, `khz`, `mhz`, `ms` and `us` as plain
`f64` numbers at priority -1. This package uses the SI spellings `Hz`, `kHz`,
`MHz`, which do not clash. For `ms` and `us` the slot picks: with both
packages, `5ms` passed where an `f64` goes is signal's number of seconds and
where a `Time` goes this package's quantity. This package's `ms` and `us` sit
at priority -2, so an untyped `let t = 5ms;` stays signal's number and adding
this package never changes signal code; write `5μs` or `ms.of(5.0)` for a
quantity there.
To leave the suffixes out, add the package with `nabi add units
--no-default-features --features ops`.

### Arrays

`Quantity<Array<f64, R>, ..>` is a whole array in one unit: `km.array(a)`,
the broadcast operators `.+ .- .* ./ .< .>`, `.at(i)`, `.sum()`,
`.values_in(u)`. A number or a scalar quantity goes on either side (`2.0 .* d`,
`d ./ 2s`, `2s .* v`, `1.0 ./ t`), and a chain of operators runs as one loop,
as on bare arrays.
`Complex<f64>` and `f32` values work with the scalar operators.

```nabi
use core::array::Array;
use units::kinds::Velocity;
use units::symbols::{m, s};

let d = m.array(Array::from_list([100.0, 200.0], [2]));
let t = s.array(Array::from_list([10.0, 25.0], [2]));
let v: Velocity<Array<f64, 1>> = d ./ t .+ 1m / 1s;   // one loop
println(v);                    // [11.0, 9.0] m/s
println(2s .* v);              // [22.0, 18.0] m
```

## Accuracy and speed

Exact factors are exact doubles (`ft` is 0.3048, `lbf` 4.4482216152605 N);
prefixes below one divide (`1 / 1e3`), so `1mm` is the double nearest 0.001.
Conversions round once or twice, about 1e-16 relative. Celsius and Fahrenheit
convert in one exact step (`20degC` is exactly `68degF`).

The dimension is only in the types, so quantity code runs as fast as bare
`f64`: scalar code compiles to the same loop, and a chain of broadcast
operators on quantity arrays fuses into one loop with no intermediate arrays,
as on bare arrays. 1e7 values, `nabi bench`, one Apple M-series core:

| Formula | bare `f64` | quantities |
|---|---|---|
| `d[i] / t[i] + v0`, summed in a counted loop | 5.66 ms | 5.66 ms |
| `d ./ t .+ v0`, arrays | 5.29 ms | 5.35 ms |
| `m .* v .* v .* 0.5`, arrays | 5.36 ms | 5.39 ms |

Logarithmic units (decibel, neper) are not in this version.

## Docs

https://packages.nabi-lang.org/packages/units/latest/docs

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at
your option.
