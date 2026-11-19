# Beam Designer

## Purpose and audience

An educational web app for students to explore beam behavior. The long-term plan includes preliminary member checks, but mechanics v1 calculates beam response only and is not a design check or a substitute for professional review.

## Platform

The app is a single-file web app served by GitHub Pages from the top-level `index.html`.

## Supported beams

- Steel W-shapes and lumber members.
- Simply supported and cantilever beams.
- Continuous beams are out of scope.

## Inputs

Each input must show its units where applicable, a valid range, its initial/default state, and an appropriate control. Numeric limits must reject non-finite and physically invalid values. Exact upper limits should be established from supported datasets and numerical constraints rather than arbitrary global caps.

| Input | Units | Valid range/options | Default | Control |
|---|---|---|---|---|
| Unit system | N/A | US customary or SI | US customary | Selector; changing units converts displayed/entered values |
| Span | Length, as selected | `0 < L <=` supported numerical limit; exact maximum TBD | 20 ft (6.096 m) in the example | Numeric field with visible units |
| Support type | N/A | Simply supported or cantilever | Simply supported | Selector |
| Material family | N/A | Steel or lumber | Steel in the example | Selector |
| Section source | N/A | Supported catalog section or custom properties | Catalog section | Selector |
| Catalog section | Section properties use the selected unit system | Entries in supported section datasets | W12x26 in the example; material-specific default TBD | Searchable selector |
| Material designation | N/A | Supported steel grade, or lumber species and grade | Unselected; required for strength checks | Selector |
| Custom section/material properties | Property-specific: e.g. length, area moment of inertia, section modulus, stress, and weight | Finite, physically valid values accepted by the selected checks; exact property schema and bounds TBD | Unset; required values must be entered before dependent checks | Material-specific fields with units and validation |
| Point loads | Force; location uses length; direction is up or down | Magnitude `>= 0`; location `0 <= x <= L`; finite values; maximum 12 loads in mechanics v1 | No point loads in the example | Repeatable load rows with magnitude, direction, and location |
| Uniform loads | Force/length; loaded interval uses length; direction is up or down | Magnitude `>= 0`; `0 <= x_start < x_end <= L`; finite values; maximum 12 loads in mechanics v1 | One 1 kip/ft load downward over the full span in the example | Repeatable load rows with magnitude, direction, and start/end locations |
| Load category | N/A | Categories supported by the selected U.S. design basis; exact list TBD | Unselected; required before code-based strength checks | Selector per load |
| Self-weight | Force/length | Derived from supported section weight; custom sections must provide weight; finite nonnegative override | Included automatically as dead load; user may override or turn off | Toggle with editable override |
| Lateral bracing / unbraced length | Length and supported restraint options | Brace locations within the span; unbraced lengths must be positive and no longer than the applicable segment/span | Unspecified; required for stability checks | Restraint selector and numeric brace-location or unbraced-length fields |
| Bearing lengths | Length | Positive values within the modeled bearing geometry; exact bounds TBD | Unspecified; required for applicable bearing checks | Numeric field at each support and other applicable bearing location |
| Design method and edition | N/A | Only the method/edition supported by the app configuration | App-selected supported basis, shown to the user | Read-only display |

Point and uniform loads are user-entered loads; the app does not determine building-level loads. Each load is classified for the planned strength combinations. The default load category is intentionally unset so the app does not silently assume dead or live load. Mechanics v1 defers load categories and combinations. Adding a point load starts with a 1-unit downward load at midspan; adding a uniform load starts with a 1-unit downward load over the full span.

The validation reference case records both self-weight settings because automatic self-weight changes moment and deflection.

Remaining full-design input decisions include numeric upper limits, material property requirements for each code check, the supported load-category list, and bearing details required at supports and concentrated loads.

For the mechanics-only v1, users enter a section label and E, I, and (when self-weight is enabled) member weight directly. The W12x26 opening example uses illustrative values only; there is no section-property catalog. Lumber is supported through user-entered properties. Load categories, material grades, bracing, and bearing inputs are deferred with code-based checks.

## Calculations (mechanics v1)

Use signed vertical loads, with upward force/reaction positive and sagging internal moment positive. A positive fixed-end reaction moment is counter-clockwise. Let `L` be span; `x` a position from the left end; `P_i` a point load at `a_i`; and `q_j` a uniform load over `[a_j, b_j]`. `R_A` and `R_B` are vertical support reactions. Self-weight, when enabled, is a downward uniform load over the full span.

For a simply supported beam:

```text
R_B = -[sum(P_i * a_i) + sum(q_j * (b_j - a_j) * (a_j + b_j) / 2)] / L
R_A = -[sum(P_i) + sum(q_j * (b_j - a_j))] - R_B
```

For a cantilever, `R_A` is the negative of the total applied vertical load. The internal root moment and external fixed-support reaction couple are:

```text
M_0 = -R_A * L
      - sum(P_i * (L - a_i))
      - sum(q_j * [(L - a_j)^2 - (L - b_j)^2] / 2)
External fixed-support reaction couple = -M_0
```

At a position `x`, calculate shear and moment by summing the reactions and the portions of loads to the left of the section:

```text
V(x) = R_A + sum(P_i for a_i < x)
       + sum(q_j * max(0, min(x, b_j) - a_j))

M(x) = M_0 + R_A * x + sum(P_i * max(0, x - a_i))
       + sum(q_j * [x * (r_j - a_j) - (r_j^2 - a_j^2) / 2])
       where r_j = min(x, b_j), and a load contributes only when x > a_j
```

For elastic deflection, use `E * I * v''(x) = M(x)` with simple-support boundary conditions `v(0) = v(L) = 0` or cantilever boundary conditions `v(0) = v'(0) = 0`. The model is prismatic, linear-elastic, small-deflection Euler-Bernoulli beam theory; it does not include shear deformation. Deflection is numerically integrated and its peak is estimated from 800 uniform sample intervals plus load locations/boundaries. Compute and display the maximum absolute shear, moment, and deflection.

Use consistent internal units (`in`, `kip`, `ksi`, `in^4`) and convert for display. `E` is elastic modulus; `I` is the area moment of inertia; `v` is vertical deflection; and `V` and `M` are internal shear and bending moment. The informational reference is `v_ref = L / 360`; it is not a pass/fail check.

## Analysis and checks

- Mechanics v1 calculates reactions, shear, moment, and elastic deflection for simply supported and cantilever beams with point loads, partial/full uniform loads, and optional self-weight.
- Mechanics v1 does not perform AISC/NDS flexure, shear, stability, bearing, load-combination, or code-compliance checks. Do not display a capacity pass/fail result.
- Connections and whole-building design, including building-level load determination, are out of scope.

## Outputs

- A beam diagram with loads and support reactions.
- Shear, moment, and deflection diagrams.
- Mechanics v1 reports maximum response values and an informational L/360 reference; it does not report member check results.
- Future code-based versions should report individual checks and an overall result without implying checks that were not performed.
- Warnings for unsupported conditions or incomplete inputs.

## Look and feel

Clean and professional, with a usable example beam loaded when the app opens.

## Validation

Include hand calculations and test cases for supported mechanics, materials, supports, loads, and unit systems. Add code-check cases only when those checks are implemented. Record input properties, units, assumptions, and expected results for every reference case.

Initial reference case: for a simply supported beam with a full-span uniform downward load of magnitude `w`, the peak magnitudes are `M_max = w * L^2 / 8` and `v_max = 5 * w * L^4 / (384 * E * I)`. With `L = 20 ft`, `w = 1 kip/ft`, `E = 29,000 ksi`, and `I = 204 in^4`, these give `M_max = 50.0 kip-ft` and `v_max` approximately 0.609 in when self-weight is disabled. With the app's default self-weight enabled at 26 lb/ft, the total downward uniform load is 1.026 kip/ft, `M_max = 51.3 kip-ft`, and `v_max` approximately 0.624 in. These W12x26 properties are illustrative, not verified design data.

## Decisions to settle before code-based checks

- Which AISC and NDS editions and which authoritative property/design-value datasets to support, including distribution/licensing constraints.
- Which load combinations and design method(s) to use for strength checks.
- How derived self-weight is classified and included in strength combinations; custom sections require user-supplied weight for automatic self-weight.
- Which stability and bearing conditions can be evaluated from the selected inputs; define warning behavior for cases that cannot be checked.
