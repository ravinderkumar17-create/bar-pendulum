# Smart Virtual Lab Design: Bar (Compound) Pendulum

## 1) Aim of the Virtual Lab
Build a realistic, student-friendly virtual experiment for a **bar (compound) pendulum** that:
- reproduces the real laboratory procedure for finding `g` and the bar’s radius of gyration,
- supports measurement uncertainty and repeat trials,
- adds a physics extension for **different surrounding media densities** (air, CO2, water, glycerin, custom fluid),
- gives instant data tables, plots, and feedback to improve scientific reasoning.

---

## 2) Theory (Physical Background)
A compound pendulum is a rigid body oscillating about a fixed horizontal axis not passing through its center of mass (COM).

For a bar pendulum:
- `m` = mass of bar,
- `l` = distance between pivot point and COM,
- `I_p` = moment of inertia about pivot,
- `I_G = m k^2` = moment of inertia about COM,
- `k` = radius of gyration about COM,
- `g` = gravitational acceleration,
- `T` = oscillation time period.

For small angular displacement `θ` (typically `< 10°`), restoring torque is

`τ ≈ -m g l θ`.

Equation of motion:

`I_p (d²θ/dt²) + m g l θ = 0`.

This is SHM with

`ω² = (m g l)/I_p`,

thus

`T = 2π √(I_p / (m g l))`.

Using parallel axis theorem:

`I_p = I_G + m l² = m(k² + l²)`.

So the usable bar pendulum period equation is:

`T = 2π √((k² + l²)/(g l))` ... (1)

or equivalently

`T² = (4π²/g) (l + k²/l)` ... (2)

---

## 3) Formula Derivation for Data Reduction

### 3.1 Linearizable form
From (2):

`T² l = (4π²/g)(l² + k²)`

Define:
- `Y = T² l`
- `X = l²`

Then:

`Y = M X + C`

where
- `M = 4π²/g`  =>  `g = 4π² / M`
- `C = (4π²/g) k² = M k²` => `k = √(C/M)`

This provides a robust linear-regression method using several pivot positions.

### 3.2 Equivalent length method
For two pivot points equidistant in period (`T1 ≈ T2`), equivalent simple pendulum length:

`L_eq = l1 + l2` (distance between conjugate points)

and

`g = 4π² L_eq / T²`.

The simulator should support both methods:
1. **Graphical linear method** (`Y vs X`),
2. **Conjugate points method** (classic real-lab procedure).

---

## 4) Extension: Surrounding Medium of Different Density
To make the lab “smart” and realistic, include damping and buoyancy effects of medium.

### 4.1 Additional physics terms
Let:
- `ρ_f` = fluid density,
- `V` = volume of bar,
- `β` = effective viscous damping coefficient,
- `I_eff` = effective inertia including added mass (optional advanced model).

Modified angular equation:

`I_eff θ¨ + β θ˙ + (m g l - ρ_f V g l_b) sinθ = 0`

For small angles:

`I_eff θ¨ + β θ˙ + K_eff θ = 0`

with

`K_eff = (m g l - ρ_f V g l_b)`.

If buoyant line of action approximates COM (`l_b ≈ l`), then effective weight term scales by `(m - ρ_f V)`.

### 4.2 Practical simulation consequences
- **Higher density medium** ⇒ lower effective restoring torque ⇒ larger `T`.
- **Higher viscosity** ⇒ faster amplitude decay and harder timing precision.
- In very dense/viscous fluid, oscillations may become overdamped (no clear periodic motion).

### 4.3 Student learning outcomes from medium variation
- Observe difference between ideal SHM and damped oscillation.
- Compare measured `g` bias in air vs fluid.
- Understand why real experiments need small-angle and low-damping assumptions.

---

## 5) Simulation Model Architecture

## 5.1 Core simulation engine
Use numerical integration (RK4 or adaptive Runge-Kutta):
- state vector: `[θ, ω]`
- dynamics:
  - `dθ/dt = ω`
  - `dω/dt = -(β/I_eff)ω - (K_eff/I_eff) sinθ`

Default step: `Δt = 0.001–0.005 s` for smooth animation.

## 5.2 Parameter inputs
### Bar properties
- length `L`
- mass `m`
- cross-section (for `V`, drag estimate)
- COM position offset (if non-uniform bar mode enabled)
- radius of gyration `k` (derived or set by geometry)

### Pivot setup
- pivot hole number / position `x_p`
- derived `l = |x_p - x_COM|`
- multiple pivots for full trial set

### Medium selection
Preset dropdown:
- Vacuum (ideal)
- Air
- CO2
- Water
- Glycerin
- Custom fluid

For each preset:
- density `ρ_f`
- dynamic viscosity `μ`
- drag model constants auto-populated.

### Initial conditions
- release angle `θ0`
- zero initial angular velocity by default.

---

## 6) GUI Features for Students (Smart Virtual Lab)

## 6.1 Student workspace layout
1. **Left panel – Setup Controls**
   - bar dimensions, mass, pivot selector
   - medium selector + advanced properties
   - angle slider + trial count
2. **Center panel – Live Animation**
   - bar oscillation with pivot marker, COM marker
   - current angle gauge
   - stopwatch and cycle counter
3. **Right panel – Data & Analysis**
   - trial table (manual + auto timing)
   - calculated `T`, `T²`, `l²`, `T²l`
   - regression results (`g`, `k`, R²)

## 6.2 Smart-assist features
- **Auto small-angle warning** when `θ0 > 10°`.
- **Damping warning** if Q-factor too low for reliable timing.
- **Procedure coach mode** guiding student through real-lab steps:
  1) select pivot,
  2) set angle,
  3) run 20 oscillations,
  4) record time,
  5) repeat 3 trials,
  6) change pivot.
- **Error bars** auto-generated from repeated trials.
- **Hint engine** (e.g., “increase oscillation count from 10 to 30 for lower timing error”).
- **Teacher mode**: hide formulas for assessment or show full derivation mode.

## 6.3 Realism options
- finite reaction time in manual stopwatch mode,
- pivot friction toggle,
- misalignment/noise option,
- bar non-uniformity option.

---

## 7) Data Table Outputs
The GUI should produce exportable CSV/Excel-ready tables.

### 7.1 Raw trial table (per pivot)
| Pivot ID | `l` (m) | Medium | Trial | Oscillations `N` | Time `t_N` (s) | `T=t_N/N` (s) |
|---|---:|---|---:|---:|---:|---:|
| P1 | 0.120 | Air | 1 | 20 | 33.24 | 1.662 |
| P1 | 0.120 | Air | 2 | 20 | 33.10 | 1.655 |
| ... | ... | ... | ... | ... | ... | ... |

### 7.2 Reduced table (mean values)
| Pivot ID | `l` (m) | `T̄` (s) | `T̄²` (s²) | `l²` (m²) | `T̄²l` (s²·m) | `σ_T` |
|---|---:|---:|---:|---:|---:|---:|
| P1 | 0.120 | 1.659 | 2.752 | 0.0144 | 0.3302 | 0.006 |
| P2 | 0.160 | 1.525 | 2.326 | 0.0256 | 0.3722 | 0.005 |

### 7.3 Analysis output table
| Medium | Slope `M` | Intercept `C` | `g` (m/s²) | `k` (m) | R² |
|---|---:|---:|---:|---:|---:|
| Air | 4.031 | 0.060 | 9.79 | 0.122 | 0.997 |
| Water | 4.188 | 0.064 | 9.42* | 0.124 | 0.991 |

`*` indicates apparent value under damped/buoyant conditions; teaching note should explain bias.

---

## 8) Graph Outputs in GUI
Mandatory graphs:
1. **`θ vs t`** (time-domain motion)
2. **`Amplitude envelope vs t`** (damping demonstration)
3. **`T²l` vs `l²`** with best-fit line (main `g` extraction)
4. **`T` vs `l`** for intuitive trend
5. **Comparison plot by medium** (Air vs Water vs Custom)

Graph features:
- hover tooltips,
- show equation + slope/intercept + R²,
- uncertainty bars toggle,
- export PNG/SVG/CSV.

---

## 9) Suggested Student Workflow (Virtual + Real Experiment Fidelity)
1. Select **Air** medium and small initial angle (`5°`).
2. Choose 6–10 pivot points on both sides of COM.
3. For each pivot, run 3 timing trials over 20 oscillations.
4. Generate reduced table and `T²l` vs `l²` graph.
5. Compute `g` and `k`, compare with standard `9.81 m/s²`.
6. Repeat complete run for Water or Glycerin medium.
7. Compare changes in period, damping, and inferred `g` bias.
8. Write observations on model assumptions and limitations.

---

## 10) Validation and Assessment Design

### 10.1 Internal validation checks
- Dimensional consistency checks for every computed column.
- Numerical stability monitor for integrator step size.
- Auto-flag if amplitude not small-angle compliant.

### 10.2 Learning assessment metrics
- correctness of table completion,
- regression quality (R² threshold),
- uncertainty interpretation,
- conceptual questions:
  - Why does damping affect period estimation?
  - Why is `g` different in dense medium in naive analysis?

---

## 11) Minimal Technical Implementation Stack (Example)
- Frontend GUI: React + Plotly/Chart.js + WebGL/Canvas animation.
- Physics engine: TypeScript or Python backend microservice.
- Data handling: client-side CSV export + optional LMS integration.
- Smart tutor rules: lightweight rule engine based on thresholds.

---

## 12) Deliverables of the Virtual Lab
- Interactive simulation with adjustable bar + pivot + medium,
- Real-lab-like stopwatch and trial recording,
- Automated analysis tables and scientific graphs,
- Formula derivation and theory tabs,
- Comparative medium-density module for advanced exploration.

This design provides a **complete smart virtual lab** that mirrors the real bar pendulum experiment while extending it with medium-density physics for deeper understanding.
