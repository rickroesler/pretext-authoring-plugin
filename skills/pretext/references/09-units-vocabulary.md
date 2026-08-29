# Unit Vocabulary

Complete list of values accepted by `@prefix` and `@base` on `<unit>` and `<per>`,
transcribed from `xsl/pretext-units.xsl` in PreTeXt core 2.51.0. An unrecognised value is
reported when you build.

`<unit>` contributes to the numerator, `<per>` to the denominator, `@exp` is an integer
power.

## Prefixes

| Name | Symbol | | Name | Symbol |
|---|---|---|---|---|
| `yocto` | y | | `deca` / `deka` | da |
| `zepto` | z | | `hecto` | h |
| `atto` | a | | `kilo` | k |
| `femto` | f | | `mega` | M |
| `pico` | p | | `giga` | G |
| `nano` | n | | `tera` | T |
| `micro` | µ | | `peta` | P |
| `milli` | m | | `exa` | E |
| `centi` | c | | `zetta` | Z |
| `deci` | d | | `yotta` | Y |

## SI base units

`ampere` A · `candela` cd · `kelvin` K · `gram` g · `meter` / `metre` m · `mole` mol ·
`second` s

Note **`gram`**, not kilogram — mass is `<unit base="gram" prefix="kilo"/>`.

## SI derived and closely related

| Group | Names |
|---|---|
| Energy, work, power, force | `joule` J, `newton` N, `pascal` Pa, `watt` W |
| Electromagnetic | `coulomb` C, `henry` H, `ohm` Ω, `siemens` S, `tesla` T, `volt` V, `electronvolt` eV, `weber` Wb |
| Frequency, catalysis | `hertz` Hz, `katal` kat |
| Radioactivity | `becquerel` Bq, `gray` Gy, `sievert` Sv |
| Temperature | `degreeCelsius` / `celsius` °C |
| Light | `lumen` lm, `lux` lx |
| Angle | `radian` rad, `steradian` sr, `degree` °, `arcminute` ′, `arcsecond` ″ |
| Time | `day` d, `hour` h, `minute` min |
| Area / volume / mass | `hectare` ha, `liter` L, `litre` l, `tonne` t |
| Other | `percent` % |

## Non-SI

These are declared to `siunitx` by PreTeXt (`\DeclareSIUnit`) because the package does not
know them.

| Group | Names |
|---|---|
| Temperature | `degreeFahrenheit` / `fahrenheit` °F |
| Weight, force | `pound` lb, `ounce` oz, `ton` T |
| Length | `foot` ft, `inch` in, `yard` yd, `mile` mi |
| Time | `millennium`, `century`, `decade`, `year` yr, `month` mo, `week` wk |
| Speed | `kilometerperhour` / `kilometreperhour` kph, `mileperhour` mph |
| Volume | `gallon` gal, `cubiccentimeter` cc, `tablespoon` tbsp, `teaspoon` tsp, `cup` c, `pint` pt, `quart` qt, `fluidounce` fl oz |
| Fuel economy | `milepergallon` mpg, `kilometerpergallon` kpg |
| Rotation | `revolution` rev, `revolutionperminute` rpm, `cycle` |
| Digital | `bit` b, `byte` B, `bitpersecond` bps, `bytepersecond` Bps |

## Common physics quantities, spelled out

```xml
<!-- 9.807 m/s^2 -->
<quantity><mag>9.807</mag><unit base="meter"/><per base="second" exp="2"/></quantity>

<!-- 1.60e-19 C  (magnitude in <m> because of the power of ten) -->
<m>1.602\times 10^{-19}</m><nbsp/><quantity><unit base="coulomb"/></quantity>

<!-- 8.314 J/(mol K) -->
<quantity><mag>8.314</mag><unit base="joule"/><per base="mole"/><per base="kelvin"/></quantity>

<!-- 1.38e-23 J/K -->
<m>1.381\times 10^{-23}</m><nbsp/><quantity><unit base="joule"/><per base="kelvin"/></quantity>

<!-- 3.0 kW -->
<quantity><mag>3.0</mag><unit base="watt" prefix="kilo"/></quantity>

<!-- 500 nm -->
<quantity><mag>500</mag><unit base="meter" prefix="nano"/></quantity>

<!-- 60 rpm -->
<quantity><mag>60</mag><unit base="revolutionperminute"/></quantity>

<!-- N m^2 / kg^2 -->
<quantity><unit base="newton"/><unit base="meter" exp="2"/>
          <per base="gram" prefix="kilo" exp="2"/></quantity>
```

## Notes

- Default LaTeX setup is `\usepackage[per-mode=fraction]{siunitx}` with
  `inter-unit-product=\cdot`. To change that you would need a custom XSL layer — it is not
  a publication-file switch.
- Units PreTeXt does not know (gauss, angstrom, atmosphere, curie, barn, parsec…) have no
  supported mechanism today. `docinfo`-declared custom units are listed as a future
  enhancement in the source. Until then, write them with `<m>\text{Å}</m>` or the like and
  accept that they will not carry unit semantics to braille.
