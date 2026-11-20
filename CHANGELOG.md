# Changelog

All notable changes to this package. Versions follow semver, independent of the Nabi toolchain.

## 0.1.0 (unreleased)

First release.

- `Quantity` with the seven SI dimension exponents in its type: a formula
  that mixes dimensions does not compile.
- `kinds` (46 named dimensions), `symbols` (about 170 units) and the 24 SI
  prefixes; unit algebra with `Unit`.
- `DynUnit` and `Measure` for units read at run time, with `UnitError`.
- `Temperature` readings on the Kelvin, Celsius, Fahrenheit and Rankine
  scales.
- CODATA `constants` with their uncertainties.
- Features `literals` (number suffixes like `3km`) and `ops` (`→`, `°`),
  both on by default; quantity arrays with the broadcast operators.
- Pure Nabi, every target including wasm. Needs Nabi 0.1 or later.
