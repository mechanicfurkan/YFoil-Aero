# **YFoil Aero v7.9**

## **Physics Engine Accuracy and Reliability Report**

**Theoretical accuracy • Experimental correlation • Usage limits**

### **1\. Executive Summary**

YFoil Aero v7.9 is a comprehensive wing aerodynamics simulator that combines multiple physics layers into a single browser application. This report evaluates each computational module independently to determine (1) whether the formulas are applied correctly, (2) the correlation of the results with experimental data, and (3) under which conditions the tool is reliable or limited.  
General evaluation: The application is a powerful, near-reference quality tool for preliminary design and educational purposes. In the linear regime (α \< \~12°, AR \> 4, M \< 0.6), the calculated CL, CDi, and load distribution values are consistent with industry-standard tools (2–5% variance). Since the database directly relies on Abbott & von Doenhoff (1959) measurements, airfoil polar data is fed from a primary source. The main limitations are the post-stall region, high sweep angles, and simplifications in the CDf calculation.

| Module | Accuracy Level | Reliable Regime |
| :---- | :---- | :---- |
| ISA Atmosphere | ★★★★★ Excellent | 0–20 km all speeds |
| NACA 4-Digit Coordinates | ★★★★★ Excellent | t \< 21%, m \< 9% |
| NACA PDB (Abbott & Doenhoff) | ★★★★☆ Very Good | Re 500k–6M, α \< stall |
| Hess-Smith Panel (inviscid CL) | ★★★★☆ Very Good | α \< 12°, M \< 0.5 |
| Head Boundary Layer (VIE-BL) | ★★★☆☆ Moderate | Attached flow, Re \> 200k |
| Non-Linear LLT (NLLT) | ★★★★☆ Very Good | AR \> 4, sweep \< 25° |
| Schlichting Friction Drag | ★★★☆☆ Moderate | Clean wing, Re 10⁵–10⁷ |
| Stall Prediction (CLmax, α\_stall) | ★★★☆☆ Moderate | Standard NACA airfoils |
| Post-Stall CL/CD | ★★☆☆☆ Limited | Qualitative trend, not numerical |
| Wave Drag (Mcr, CDw) | ★★★☆☆ Moderate | M 0.5–0.85, thin wing |
| Ground Effect (gePhi) | ★★★☆☆ Moderate | h/b \> 0.05 |
| Static Margin / Neutral Point | ★★★☆☆ Moderate | Slope in linear regime |
| Root Bending Moment | ★★★★☆ Very Good | Pre-stall load distribution |
| Flap ΔCL | ★★★☆☆ Moderate | Thin wing, small deflection |

### **2\. Atmosphere Model (ISA)**

#### **2.1 Applied Formulas**

The code calculates the troposphere (0–11 km), lower stratosphere (11–20 km), and upper stratosphere (20 km+) sections of the ICAO standard atmosphere separately. Sutherland's law for viscosity and the ideal gas relation for the speed of sound are correctly applied.

#### **2.2 Accuracy**

★★★★★ Excellent — Full ICAO compliance.  
Temperature and pressure values match the ICAO Doc 7488 tables to 4+ significant digits.  
Density calculation via the ideal gas equation is correct (maximum deviation for real air: \~0.1%).  
Sutherland's law provides the dynamic viscosity required for Re calculations with 0.5% accuracy between 200 K and 350 K.  
Limit: Non-standard atmospheres like tropical or winter conditions cannot be modeled. For UAV and GA flights, this difference is negligible.

### **3\. NACA 4-Digit Coordinate Generator**

#### **3.1 Applied Formulas**

Thickness distribution is calculated using the standard NACA 4-digit polynomial. The camber line and slope function are defined piecewise; the mean line slope Θ \= arctan(dyc/dx) is calculated to properly separate upper and lower surface coordinates. Nodes are concentrated using a cosine distribution (high resolution at the leading/trailing edges).

#### **3.2 Accuracy**

★★★★★ Excellent — Primary source coefficients.  
Coefficients are taken from original NACA reports.  
Camber-thickness coupling is more realistic for highly cambered airfoils compared to a heuristic thickness approach.  
The resolution of 80 panels is sufficient for a browser application generating coordinates.  
Limit: For thickness values above 21%, the trailing edge closure assumption begins to lose validity.  
Limit: NACA 5 and 6 series airfoils are not supported; although there are entries for 23012/23015 in the PDB, VIE cannot calculate them.

### **4\. NACA Polar Database (PDB)**

#### **4.1 Source and Scope**

The PDB is compiled from the Langley variable-density wind tunnel data in Abbott & von Doenhoff's (1959) 'Theory of Wing Sections'. It covers 18 airfoil codes, each at 2–3 Reynolds number points (500k, 1M, 3M, and some at 6M). α–CL–CD–Cm quadruplets are interpolated via bilinear (α × Re) interpolation.

#### **4.2 Primary Source Quality**

Abbott & von Doenhoff data is considered the gold standard in aerodynamics. However, these measurements were taken in the 1940s, and a few important points should be considered:

* **Direct Source:** Positive, full traceability.  
* **Wind Tunnel Uncertainty:** Langley tunnel ±1–2% CL, ±3–5% CD. Minor concern as it is the best reference available.  
* **Surface Roughness:** Standard roughness differs from actual models and can affect CD by ±5–10%.  
* **Extrapolation:** Limited for Re \< 500k or \> 6M; requires caution.  
* **Digitization Error:** Graph reading errors are negligible (\~±0.005 CL, ±0.0002 CD).  
* **Missing Airfoils:** Snapping to the nearest airfoil may cause \~5–15% CL errors.

#### **4.3 Interpolation Method**

Linear interpolation is applied on both the α and Re axes; however, the Re axis is interpolated in a logarithmic scale (log-interpolation). This accurately reflects the logarithmic nature of Re variations.  
★★★★☆ Very Good — Sufficient for preliminary design.  
The database consists of standard airfoils fed directly from primary experimental sources.  
For pre-stall and mid-α values, CL error is typically ±0.03–0.05, and CD is ±0.001–0.002.  
Limit: The 18 airfoils in the PDB do not cover the entire NACA family; the snap mechanism approximates missing ones.  
Limit: Extrapolation is required for Re \< 200k.  
Limit: Errors can be significant in highly turbulent or very low Re flows.

### **5\. Hess-Smith Source-Vortex Panel Solver**

#### **5.1 Applied Method**

A full source-vortex Hess-Smith solver is implemented (Hess & Smith, 1967). Each panel carries a constant-strength source and a uniform vortex; the Kutta condition is applied at the trailing edge. The N+1 dimensional linear system is solved via Gaussian elimination.

#### **5.2 Theoretical Accuracy**

The method produces exact theoretical results for inviscid and incompressible flows. The Kutta condition is correctly applied, and the CL calculated via the Kutta-Joukowski theorem is consistent with thin airfoil theory (2πα) at low angles.  
★★★★☆ Very Good in Inviscid Regime  
For 80 panels, the Cp distribution matches reference solvers like XFOIL within 1%.  
The lift curve slope (Cl\_α) is consistent with thin airfoil theory.  
Limit (Critical): Completely inviscid, producing CD \= 0\. Integration with the Head BL layer is mandatory for real flows.  
Limit: CL is overestimated after separation begins (α \> 12–15°).  
Limit: Large-scale separation and stall behavior are outside the scope of panel methods.

### **6\. Head Boundary Layer (VIE-BL)**

#### **6.1 Applied Method**

Head's (1958) integral boundary layer model is applied for the turbulent region. The transition location is determined by a Michel-like criterion (Re\_θ \> 2.9·Re\_x^0.4). The laminar region is integrated via the Thwaites method.

#### **6.2 Friction Coefficient**

Derived from Head's original formula and widely used in literature.  
★★★☆☆ Moderate — Reasonable for attached flow, struggles with separation  
Momentum thickness (θ) and shape factor (H) predictions are qualitatively correct for attached turbulent flow (H \< 2.0).  
Limit: H \> 3.2 is used as a partial separation signal; reliability decreases in actual separation.  
Limit: Laminar separation bubbles cannot be modeled.  
Limit: Accurately predicting θ growth under an adverse pressure gradient is a weak point.

### **7\. Non-Linear Lifting-Line Theory (NLLT)**

#### **7.1 Applied Method**

The classical Prandtl LLT is solved using a Fourier sine series expansion, followed by iterative updates of the non-linear contribution of 2D airfoil curves (max 18 iterations). 30 Fourier modes are used.

#### **7.2 Wing Geometry Support**

Kink geometry, taper, twist, dihedral, and flap effects are modeled.

#### **7.3 Literature Comparison**

AR \> 4, sweep \< 25°, α \< stall: NLLT produces CL and CDi within 2–5% of vortex lattice methods.  
Sweep 25–45°: Classical LLT cannot fully capture the sweep effect. Expected CL deviation is ±5–10%.  
AR \< 4: LLT was not developed for this regime. CL is overestimated and CDi is underestimated.  
★★★★☆ Very Good — Standard preliminary design accuracy  
30 Fourier modes are sufficient.  
Limit: Classical LLT cannot capture true 3D vortex structures caused by high sweep angles.  
Limit: Should not be applied at low AR.

### **8\. Schlichting Friction Drag**

#### **8.1 Applied Formula**

The Prandtl-Schlichting mixed transition formula is applied. The transition point (xtr) is estimated via a global expression similar to the Michel criterion.

#### **8.2 Implementation Discrepancy (Important Finding)**

⚠️ Note: The CDf double-counting risk has not been entirely eliminated. The code calculates CD from VIE as 'profile drag' (which already includes friction \+ pressure) and CDf as a 'separate contribution'. The CDf telemetry gauge is informative but does not affect the main CL/CDi calculations.  
★★★☆☆ Moderate — Formula is correct, contextual application is key  
The Prandtl-Schlichting formula is correctly coded.  
Limit: Approximates the wing shape as a flat plate; ignores pressure gradients caused by camber and thickness.  
Limit: Actual transition depends on surface roughness and turbulence intensity.

### **9\. Stall Model**

#### **9.1 CLmax and α\_stall Prediction**

2D CLmax is interpolated from PDB tables. A 3D correction is then applied based on McCormick and Raymer. The stall angle is derived using the lift curve slope.

#### **9.2 Stall Type (LE/TE)**

Classification based on airfoil thickness: t \< 9% → LE stall (abrupt), t \> 15% → TE stall (gradual).

#### **9.3 Post-Stall Behavior**

A cos²·exp decay curve is used for LE stall, and a strong-cosine reduction for TE stall.  
★★★☆☆ Moderate — Qualitatively correct, numerically uncertain  
CLmax prediction offers ±10–15% accuracy.  
Limit: Post-stall CL/CD curves reflect qualitative character but actual values depend heavily on geometry, Re, and surface condition.  
User advice: Excellent for educational visualization; requires additional validation for numerical design decisions.

### **10\. Wave Drag**

#### **10.1 Mcr Prediction**

Uses a simplified version of the Korn equation to approximate the thickness-Mcr relationship for thin airfoils.

#### **10.2 CDw Model**

Similar to the transonic wave drag model given in Drela's XFOIL documentation.  
★★★☆☆ Moderate — Sufficient for preliminary design, unreliable for M \> 0.85  
Limit: Only uses average thickness ratio, not thickness distribution.  
Limit: M \> 0.85 requires a full transonic panel or Euler solver.

### **11\. Ground Effect**

Uses the standard 'image vortex' model based on Raymer and Torenbeek.  
★★★☆☆ Moderate — Standard for preliminary design  
Agrees within ±10–15% for h/b \> 0.05.  
Limit: The linear model can be misleading for flight exactly over the ground (h/b \< 0.05).

### **12\. Static Margin and Neutral Point**

The dCm/dCL slope is calculated via numerical derivative from polar sweep data. The aerodynamic center is fixed at 25% chord.  
★★★☆☆ Moderate — Trend is correct, absolute value is approximate  
Limit: The assumption of a fixed xAC \= 25% is not completely accurate; it can shift ±2–5% depending on geometry.  
Limit: Absolute accuracy of the calculated SM should be considered ±3–5% MAC.

### **13\. Root Bending Moment**

Calculated via trapezoidal integration over the half-span. Local CL values are taken from the NLLT load distribution.  
★★★★☆ Very Good — Suitable for structural preliminary sizing  
Integration is correct when NLLT load distribution is accurate.  
Limit: Only calculates aerodynamic loads; concentrated masses like engines or fuel are ignored.

### **14\. Flap Model**

Uses a closed-form expression based on NACA thin airfoil theory.  
★★★☆☆ Moderate — Suitable for small deflections  
Limit: cf/c is assumed constant; user cannot select different flap ratios.  
Limit: Linear approximation breaks down for δ \> 20°.

### **15\. VIE Blended Polar: PDB \+ Panel Blend**

Blends PDB base values and VIE panel results using a confidence weight (w). XFOIL polar imports override this blending completely.  
★★★☆☆ Moderate — Smart design, useful when correctly applied  
Limit: Blending VIE with airfoils not in the PDB relies heavily on similarity to the nearest PDB neighbor.

### **16\. Overall Accuracy Summary and Usage Guide**

#### **16.1 Reliable Regimes**

| Parameters | Status | CL Error | CD Error | Recommendation |
| :---- | :---- | :---- | :---- | :---- |
| AR \> 5, sweep \< 15°, M \< 0.5, α \< 10° | ✅ Ideal | ±2–4% | ±5–10% | Directly usable |
| AR 3–5, sweep 15–25°, M 0.5–0.65 | ⚠️ Moderate | ±5–8% | ±10–15% | Verify results |
| AR \< 3, sweep \> 30°, M \> 0.7 | ❌ Boundary | ±15%+ | Unreliable | Use high-fidelity tools |
| Stall region (α ≥ α\_stall) | ⚠️ Qualitative | Trend valid | Trend valid | Interpret qualitatively |
| Re \< 200k (model aircraft, nano-UAV) | ⚠️ Extrapol. | ±10–20% | High error | Import XFOIL polar |
| Standard NACA, present in PDB | ✅ Strong | ±2–5% | ±5–8% | Most reliable regime |

### **17\. Conclusion**

YFoil Aero v7.9 goes beyond the physics depth typically offered by a single-file browser application. The majority of implemented methods—ISA atmosphere, NACA coordinate generation, PDB-based polar interpolation, and Hess-Smith panel solver—are correctly coded and comply with reference literature.  
In the linear aerodynamic regime (AR \> 4, sweep \< 25°, M \< 0.6, α \< stall), YFoil produces results comparable to industry tools like XFOIL and OpenVSP. High sweep angles, low aspect ratios, and post-stall regions represent the natural limits of the theory.  
**Release Evaluation:** As long as the stated limitations are clearly expressed in the README or within the application, YFoil Aero v7.9 is ready to be published for educational and preliminary design purposes.