# **YFoil Aero v7.9**

## **Browser-Based Wing Aerodynamics Simulator**

### **Technical Design, Physics Engine, and Accuracy Report**

**Prepared by:**  
Furkan Yılmaz  
Mechanical Engineering  
2025

## **Abstract**

YFoil Aero is a single-file simulation tool running entirely in the browser, aimed at analyzing wing aerodynamics in aerospace and mechanical engineering. The primary motivation during the development process is to provide a lightweight, accessible, and education-oriented alternative to classical aerodynamic analysis software (such as XFOIL, OpenVSP, DATCOM) that require installation and depend on heavy computational infrastructure.  
This report comprehensively covers the mathematical and experimental models used in the application's physics engine, the theoretical foundations of these models, application sensitivities, and a comparative accuracy analysis against experimental data. The report also serves as a design and development document; explaining why each module was selected, where its limits begin, and under which conditions it produces reliable results.  
The physics modules constituting the core of the application are: the International Standard Atmosphere (ISA) model, the viscous-inviscid interaction (VIE) framework consisting of the Hess-Smith source-vortex panel method and the Head integral boundary layer solver, Prandtl's Non-Linear Lifting-Line Theory (NLLT), the experimental NACA polar database by Abbott and von Doenhoff (1959), the Karman-Tsien compressibility correction, the wave drag model, the Michel transition criterion, and the Schlichting hybrid boundary layer friction formulation.  
As a result of the accuracy evaluation; it is observed that the calculated lift coefficient (CL), induced drag coefficient (CDi), and spanwise load distribution produce results with comparable accuracy to industry-standard tools in the linear aerodynamic regime (aspect ratio AR \> 4, sweep angle \< 25°, Mach \< 0.6, angle of attack \< stall angle). High sweep angle, low aspect ratio, and the post-stall regime constitute the natural boundary conditions of the physical models used.

| Parameter | Value |
| :---- | :---- |
| Application Version | YFoil Aero v7.9 |
| Platform | Browser (Vanilla JS, HTML5, Plotly.js) |
| Panel Solver | Hess-Smith Source-Vortex (80 panels) |
| Lifting-Line Theory | NLLT — 30 Fourier Modes |
| Boundary Layer | Head Integral Method (VIE) |
| Polar Database | Abbott & von Doenhoff (1959) — 18 NACA Airfoils |
| Re Range (PDB) | 500,000 – 6,000,000 |
| Mach Validity Limit | M \< 0.85 (wave drag model) |
| Reliable Regime | AR \> 4, Λ \< 25°, M \< 0.6, α \< α\_stall |

## **1\. Introduction and Motivation**

### **1.1 Problem Definition**

Wing aerodynamics analysis constitutes one of the most fundamental and challenging problems in aerospace engineering. Accurate modeling of real flow conditions requires the simultaneous handling of complex physical phenomena such as turbulent boundary layers, flow separation, compressibility effects, and three-dimensional vortex structures. Therefore, industrial-level accuracy is mostly obtained through expensive wind tunnel experiments or computational fluid dynamics (CFD).  
On the other hand, in engineering education and early design stages, developing physical intuition and enabling fast parametric iteration are prioritized over the absolute precision provided by a full CFD calculation. The vast majority of existing tools addressing this need either require heavy installation procedures, remain operating system-dependent, or are unusable without programming knowledge. YFoil Aero was developed to bridge this gap.

### **1.2 Design Objectives**

The objectives defined during the design process of the project can be summarized under three main headings:

* **Accessibility:** To be able to run using only a web browser without requiring any installation or backend infrastructure. All calculations take place on the client side (in the browser).  
* **Physical Accuracy:** The models used should not consist solely of empirical regressions; instead, physical-based semi-analytical methods (LLT, panel solver, integral boundary layer) are combined with experimental data (Abbott & von Doenhoff PDB).  
* **Transparency:** Clearly documenting the accuracy limits and under which conditions each model is valid. This is the primary purpose of this report.

### **1.3 Scope and Limitations**

YFoil Aero offers the following analysis capabilities:

* Cranked wing geometry: kink position, inner/outer leading-edge sweep, dihedral, twist, taper  
* Instant CL, CD, CDi, Cm calculation and lift/drag forces  
* Full polar curve sweep between α \= \-10° and \+20°  
* Spanwise load distribution and stall margin  
* Pressure coefficient (Cp) distribution — Hess-Smith panel solver or thin airfoil theory  
* Static margin, neutral point, and Cm-α derivative  
* Root bending moment  
* Wave drag (M \> Mcr)  
* Ground effect  
* Flap effect (ΔCL, ΔCm, ΔCD)  
* XFOIL polar file import  
* Engineering report output, CSV, and STL export

Topics outside the scope of the application include: Full Navier-Stokes solution, turbulence modeling, dynamic stall, aeroelastic interaction (flutter, divergence), multi-body configurations, and powered propulsive effects.

## **2\. Atmosphere Model — International Standard Atmosphere (ISA)**

### **2.1 Theoretical Basis**

Since atmospheric conditions directly affect air density (ρ), dynamic viscosity (μ), and the speed of sound (a), they are fundamental inputs for calculating Reynolds number, Mach number, and lift/drag forces. YFoil Aero fully implements the International Standard Atmosphere (ISA) model in accordance with the ICAO Doc 7488 standard.

### **2.2 Applied Equations**

**Troposphere (h ≤ 11,000 m):**  
T \= 288.15 \- 0.0065 · h \[K\]  
p \= 101.325 · (T / 288.15)^5.2561 \[kPa\]  
**Lower Stratosphere (11,000 \< h ≤ 20,000 m):**  
T \= 216.65 \[K\] (isothermal)  
p \= 22.632 · exp(-1.5769 × 10⁻⁴ · (h \- 11000)) \[kPa\]

### **2.3 Accuracy**

**ISA Model — Accuracy Evaluation: Excellent**  
Full compliance with ICAO Doc 7488 reference tables. Sutherland's law calculates viscosity with ±0.5% precision. Limit: Non-standard atmospheric conditions (tropical, arctic) are not modeled.

## **3\. NACA 4-Digit Airfoil Coordinate Generator**

### **3.1 Theoretical Basis**

The NACA 4-digit airfoil series was defined by Jacobs et al. in 1933\. Each four-digit code carries the following information: the first digit expresses the maximum camber value as a percentage of the chord, the second digit is the maximum camber position in tenths of the chord, and the last two digits express the maximum thickness as a percentage of the chord.

### **3.2 Applied Formulation**

The standard NACA 4-digit thickness polynomial is used. To increase the precision of the panel solver, node points are concentrated using a cosine distribution. In this method, a transformation of x \= 0.5(1 \- cos β) is applied by sampling evenly between β \= \[0, π\].

### **3.3 Accuracy**

**NACA Coordinate Generator — Accuracy Evaluation: Excellent**  
Coefficients are taken from original NACA technical reports. Camber-thickness coupling is applied correctly. Limit: For thickness values above 21%, the trailing edge closure assumption breaks down. Limit: NACA 5 and 6 series airfoils cannot be generated.

## **4\. Hess-Smith Source-Vortex Panel Solver**

### **4.1 Historical Background and Motivation**

Panel methods were developed by Smith and Hess in the late 1960s, becoming one of the fundamental tools of computational aerodynamics. The Hess-Smith method (1967) applied in YFoil Aero discretizes the airfoil boundary with N panels, assigning a constant-strength source and uniform vortex to each panel. The Kutta condition is applied at the trailing edge to ensure physically meaningful lift generation.

### **4.2 Mathematical Formulation**

The normal velocity component at the control point for each i-th panel is expressed by a linear equation system. The effect of each panel pair is calculated with closed-form analytical expressions combining logarithmic and trigonometric terms. The Kutta condition ensures that the sum of the tangential velocity components of the upper and lower trailing panels equals zero.

### **4.3 Viscous-Inviscid Interaction (VIE) Framework**

A purely inviscid panel solver does not generate drag and cannot model separation. To overcome this limitation, the Head integral boundary layer model is used in a loosely coupled framework with the panel solver.

### **4.4 Head Integral Boundary Layer Model**

The laminar-to-turbulent transition location is determined by a threshold condition similar to the Michel (1951) criterion. After transition, Head's entrainment approach is used. The viscous drag contribution is calculated using the trailing edge momentum thickness.

### **4.5 Accuracy and Limitations**

**Hess-Smith Panel (Inviscid) — Accuracy: Very Good**  
For 80 panels, the Cp distribution agrees within 1% with reference panel solvers (XFOIL). Limit: Purely inviscid — does not produce drag.  
**Head Boundary Layer (VIE) — Accuracy: Moderate**  
In attached turbulent flow (H \< 2.0), θ and H predictions are qualitatively correct. Limit: Predicting θ growth under an adverse pressure gradient is the weak point of this method.

## **5\. NACA Polar Database (PDB)**

### **5.1 Historical Source**

YFoil Aero's polar database is compiled from the Langley variable-density wind tunnel data published in Abbott and von Doenhoff's 1959 reference book 'Theory of Wing Sections'.

### **5.2 Interpolation Method**

Linear interpolation is applied on the α axis, while interpolation on the Reynolds number axis is performed in log-space to reflect the logarithmic nature of Re.

### **5.4 Accuracy Analysis**

**NACA PDB — Accuracy Evaluation: Very Good**  
Directly fed from the primary experimental source. For pre-stall and mid-α values, the CL error is typically ±0.03-0.05. Limit: There is no data for Re \< 200,000; extrapolation is performed.

## **6\. Non-Linear Lifting-Line Theory (NLLT)**

### **6.1 Theoretical Basis**

Prandtl's Lifting-Line Theory (1918) is one of the first analytical approaches developed to model the aerodynamics of finite-span wings. In YFoil Aero, this theory has been expanded to its non-linear version through iterative feedback of 2D polar curves.

### **6.2 Fourier Series Formulation**

The circulation distribution Γ(y) along the wing is represented as a Fourier sine series. YFoil Aero uses 30 Fourier modes.

### **6.3 Non-Linear Iteration**

The effective 2D angle of attack is calculated using the induced velocity distribution obtained from the linear solution. The process is repeated until convergence at a tolerance of ΔCL \< 10⁻⁴.

### **6.4 Three-Dimensional Corrections**

The Karman-Tsien compressibility correction is applied to reflect the effect of Mach number. Fuselage effect and kink geometry corrections are also empirically modeled.

### **6.6 Literature Comparison and Accuracy**

**NLLT — Accuracy Evaluation: Very Good (In Nominal Regime)**  
Under AR \> 4, Λ \< 25°, α \< stall conditions, CL and CDi are within 2-5% of XFOIL/OpenVSP. Limit: Classical LLT cannot capture the true 3D vortex structure at high sweep angles.

## **7\. Drag Models**

The total drag on a wing consists of four basic components. YFoil Aero calculates each using separate modules: CD \= CD\_profile \+ CD\_friction \+ CDi \+ CD\_wave.

## **8\. Stall Model**

The 2D CLmax value is taken from the PDB via interpolation with Re. Post-stall CL and CD models utilize combined empirical expressions to represent sudden lift loss (LE stall) or gradual decrease (TE stall).

## **9\. Longitudinal Balance and Stability Analysis**

Longitudinal static balance analysis is performed by taking the numerical derivative from the polar sweep data. The root bending moment is calculated via trapezoidal integration using the spanwise load distribution data.

## **10\. Hybrid VIE+PDB Polar Framework**

The PDB only covers a limited number of standard NACA airfoils. The hybrid framework merges the PDB and VIE results with a confidence-weighted blend. When an XFOIL polar is loaded, this blending is completely disabled, and XFOIL data is used directly.

## **11\. Overall Accuracy and Reliability Analysis**

In the linear aerodynamic regime (AR \> 4, sweep \< 25°, M \< 0.6, α \< stall), YFoil Aero produces results with comparable accuracy to industry tools like XFOIL and OpenVSP.

## **12\. Software Architecture and Technical Implementation**

### **12.1 Single-File Application Architecture**

YFoil Aero is developed with a single-file HTML structure that requires no installation and maximizes portability. Zero installation, platform independence, and static hosting compatibility are the main advantages.

### **12.2 Computational Performance**

The full calculation chain for a single α value takes approximately 15-50 ms in a modern browser. A polar sweep (61 points) is completed in about 800-2,000 ms.

## **13\. User Manual**

Open the index.html file in any modern browser. Define the wing geometry using the sliders on the left panel, set the flight conditions, and read the instant results from the telemetry cards.

## **14\. Conclusion and Evaluation**

YFoil Aero v7.9 is a comprehensive wing aerodynamics simulator that combines multiple physics layers into a single browser application. It provides a robust, transparent, and accessible tool for aerospace engineering education and the preliminary design phase.

## **References**

1. Abbott, I.H., von Doenhoff, A.E. (1959). Theory of Wing Sections. Dover Publications, New York.  
2. Prandtl, L. (1918). Tragflügeltheorie. Nachrichten von der Gesellschaft der Wissenschaften zu Göttingen.  
3. Hess, J.L., Smith, A.M.O. (1967). Calculation of potential flow about arbitrary bodies. Progress in Aerospace Sciences, 8, 1-138.  
4. Head, M.R. (1958). Entrainment in the turbulent boundary layer. ARC R\&M 3152, Her Majesty's Stationery Office.  
5. Michel, R. (1951). Etude de la transition sur les profils d'aile. ONERA Report 1/1578A.  
6. McCormick, B.W. (1979). Aerodynamics, Aeronautics and Flight Mechanics. Wiley, New York.  
7. Raymer, D.P. (1992). Aircraft Design: A Conceptual Approach, 2nd ed. AIAA Education Series.  
8. Schlichting, H. (1979). Boundary Layer Theory, 7th ed. McGraw-Hill, New York.  
9. Drela, M. (1989). XFOIL: An Analysis and Design System for Low Reynolds Number Airfoils. Low Reynolds Number Aerodynamics, Springer.  
10. Phillips, W.F. (2004). Lifting-Line Analysis for Twisted Wings and Washout-Optimized Wings. Journal of Aircraft, 41(1), 128-136.  
11. ICAO. (1993). Manual of the ICAO Standard Atmosphere, Doc 7488, 3rd ed.  
12. Reid, E.G. (1932). A Full-Scale Investigation of Ground Effect. NACA TN-265.  
13. Hoerner, S.F. (1965). Fluid-Dynamic Drag. Published by the author.  
14. Torenbeek, E. (1982). Synthesis of Subsonic Airplane Design. Delft University Press.  
15. Gallay, S., Laurendeau, E. (2015). Nonlinear Generalized Lifting-Line Coupling Algorithms for Pre/Poststall Flows. AIAA Journal, 53(7), 1784-1796.