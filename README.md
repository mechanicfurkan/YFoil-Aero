# YFoil Aero

**Browser-based wing aerodynamics simulation tool.**  
Analyzes swept wing geometries using Prandtl's Lifting-Line Theory, a viscous-inviscid panel solver, and a real NACA airfoil database.

> **Purpose:** Education and preliminary design. It should not be used as the sole basis for certification or flight-critical engineering decisions.
![YFoil Aero Demo](Foilteest.gif)
---

## Features

Define a wing geometry (span, chord, sweep, twist, NACA airfoil), and the application instantly calculates:

- **CL, CD, CDi, Cm** — lift, drag, induced drag, pitching moment
- **L/D ratio** — aerodynamic efficiency
- **Spanwise load distribution** — spanwise CL curve
- **Pressure coefficient distribution** — Cp plot
- **Stall angle and stall type** — leading edge (LE) vs. trailing edge (TE) separation
- **Static margin and neutral point** — longitudinal stability analysis
- **Root bending moment** — structural preliminary sizing
- **Wave drag** — for Mach > 0.5 regimes
- **Ground effect** — via AGL height parameter

By importing an XFOIL polar file (.txt/.dat), you can use custom airfoils outside the built-in NACA database.

---

## Physics Engine

| Module | Method | Reference |
|---|---|---|
| Lifting-line theory | Nonlinear LLT (30 Fourier modes) | Prandtl (1918), McCormick (1979) |
| Panel solver | Hess-Smith source-vortex method + Head's boundary layer | Hess & Smith (1967) |
| Airfoil database | NACA experimental data (Re interpolation) | Abbott & von Doenhoff (1959) |
| Transition prediction | Michel's criterion | Michel (1951) |
| Wave drag | Kármán-Tsien correction + Mcr model | Drela, XFOIL documentation |
| Atmosphere model | ISA standard (0–20 km) | ICAO Doc 7488 |
| Flap effect | NACA ΔCL_δ method | Raymer (1992) |
| Stall model | LE/TE stall type distinction, post-stall cos²·exp transition | McCormick (1979) |

---

## Accuracy and Limitations

This tool was developed for preliminary design and educational purposes. The results are based on **theoretical calculations**.

**Most reliable regime:**
- Aspect ratio (AR) > 4
- Sweep angle < 25°
- Mach < 0.6
- Angle of attack < stall angle

Under these conditions, the CL error is typically ±3–5%, and the CDi error is ±5–10%; these values are consistent with standard preliminary aerospace engineering design margins.

**Edge cases to be aware of:**
- AR < 4 or high sweep angles (> 30°): Exceeds the theoretical limits of LLT, causing the error to increase.
- Post-stall region: Trends are accurate, but numerical values are not precise.
- Re < 200,000 (model aircraft, nano-UAVs): The database does not cover this regime; importing XFOIL polars is recommended.

For detailed accuracy analysis: [`YFoil_Dogruluk_Raporu.docx`](./YFoil_Dogruluk_Raporu.docx)

---

## Usage

No installation required. It is a single HTML application:

```text
Open the YFoil_Aero.html file in your web browser.
```

Live demo: **[https://mechanicfurkan.github.io/YFoil-Aero/](https://mechanicfurkan.github.io/YFoil-Aero/)**

---

## Version History

| Version | Description |
|---|---|
| v5.x | First working prototype, DATCOM-based lift model |
| v6.0 | Fixed CD double counting bug, post-stall cos²·exp curve |
| v7.0 | Transition to LLT, actual NACA PDB, bilinear Re+α interpolation |
| v7.5 | VIE viscous-inviscid panel solver, 2D wind tunnel view |
| v7.9 | Simplified panel system, eliminated CDf double counting, al0 caching |
| v8.x | UI branding enhancements, dynamic dark mode for 3D wing visualization, aero performance optimizations achieving stable 60 FPS (debouncing & geometry caching), and robust external airfoil import fixes. |

---

## License

**Creative Commons Attribution-NonCommercial 4.0 (CC BY-NC 4.0)**

You are free to: Use · Share · Adapt  
Under the following terms: Attribution is required · Non-commercial use only

Full text: [LICENSE](./LICENSE)

---

## Author

This project was developed by a Mechanical Engineering student at Yildiz Technical University to provide an intuitive, reference-quality tool for young aerospace engineers. Created by M. Furkan YILMAZ.

Reference literature: Abbott & von Doenhoff (1959) · McCormick (1979) · Raymer (1992) · Drela XFOIL
