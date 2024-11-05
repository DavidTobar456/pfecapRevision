# Detailed Explanation of the Preisach Ferroelectric Capacitor Model (PFECAP)

This document provides a detailed explanation of the Preisach Ferroelectric Capacitor (PFECAP) model, as coded in the Verilog-A file. This model is based on several key pieces of literature, including works by Bo Jiang et al. (1997) on computationally efficient ferroelectric capacitor models, and research by K. Ni et al. (2018) on compact modeling for ferroelectric FETs. The code implements the Preisach hysteresis model for ferroelectric capacitors, which is commonly used to simulate nonlinear switching and history-dependent behaviors.

## Overview

The PFECAP model simulates the behavior of ferroelectric capacitors, specifically focusing on their hysteresis behavior. Ferroelectric materials exhibit a non-linear relationship between charge and voltage, and they have a history-dependent memory effect. This model aims to capture these properties using an efficient implementation that makes it suitable for circuit simulations.

### Key Parameters

- `tFE`: Thickness of the ferroelectric layer (m).
- `Ec`: Coercive field (V/m), which represents the electric field required to switch the polarization of the ferroelectric material.
- `epsFE_r`: Relative permittivity of the ferroelectric material.
- `Qs`: Maximum polarization charge (Coul/m²).
- `a`: Slope adjustment parameter for the hyperbolic tangent function.
- `rho`: Parameter used for modeling the delay effect in ferroelectric switching, based on the Landau-Khalatnikov (L-K) equation.

## Analog Functions

The code contains several key analog functions that model various aspects of the ferroelectric capacitor. Below, we discuss each function in detail and explain their purpose, along with references to the literature where applicable.

### 1. `F(Volt, dir)` - Ferro Loop Function

*From line 75-85*

This function calculates the polarization hysteresis loop based on the input voltage (`Volt`) and the direction of polarization switching (`dir`). It uses a hyperbolic tangent function to capture the non-linear behavior of the ferroelectric capacitor, as described by Bo Jiang et al. (1997).

- **Formula**:

  ```
  F = Qs * tanh(a * (Volt - dir * Ec * tFE));
  ```
- **Explanation**: The hyperbolic tangent function is used because it provides a good fit for the saturation characteristics of ferroelectric capacitors. The parameter `a` adjusts the slope of the function, which controls how steeply the polarization switches as voltage increases.

### 2. `calc_m(a_P, b_P, a_F, b_F)` - Multiplier Calculation

*From line 87-107*

This function calculates the slope (`m`) of the linear segment between two points on the polarization curve. This is an important part of determining the behavior of minor hysteresis loops within the Preisach model.

- **Formula**:

  ```
  calc_m = DP / DF;
  ```

  where `DP = a_P - b_P` and `DF = a_F - b_F`.
- **Explanation**: The slope `m` represents how the polarization changes between two different voltage points. This value is used to describe the linear segments connecting turning points in the hysteresis loop, allowing the model to accurately capture minor loop trajectories.

### 3. `calc_b(a_P, b_P, a_F, b_F)` - Offset Calculation

*From line 109-129*

This function calculates the offset (`b`) for the linear segment on the polarization curve. Together with `calc_m()`, it helps to accurately define the trajectories of minor loops within the Preisach model.

- **Formula**:

  ```
  calc_b = (b_P * a_F - a_P * b_F) / (a_F - b_F);
  ```
- **Explanation**: The offset `b` is used along with the slope `m` to determine the complete equation of the polarization for a given segment. This allows the model to transition smoothly between different parts of the hysteresis loop.

### 4. `P(Volt, dir, m, b)` - Scaled Ferro Loop Function

*From line 131-140*

This function calculates the polarization (`P`) for a given input voltage. It uses the slope (`m`) and offset (`b`) calculated from the previous functions.

- **Formula**:

  ```
  P = F(Volt, dir) * m + b;
  ```
- **Explanation**: The function `P()` models the polarization response within the context of the overall hysteresis behavior. This approach follows the Preisach model methodology, where the overall hysteresis loop is built from multiple smaller segments.

### 5. `Q(Volt, dir, m, b)` - Final Ferroelectric Charge Loop

*From line 142-152*

This function calculates the total charge (`Q`) of the ferroelectric capacitor. It combines the polarization component with the capacitive contribution to produce the overall charge response.

- **Formula**:

  ```
  Q = P(Volt, dir, m, b) + epsFE_r * EPS0 * Volt / tFE;
  ```
- **Explanation**: This function adds the capacitive component (based on the relative permittivity and thickness of the ferroelectric layer) to the polarization component to determine the overall charge response. This follows the standard approach for modeling ferroelectric capacitors, where both the nonlinear polarization and linear dielectric contributions are included.

### 6. `dF(Volt, dir)` - Derivative of Ferro Loop Function

*From line 154-165*

This function calculates the derivative of the function `F()` with respect to the input voltage. This derivative is used as part of Newton's method for determining the coercive voltage.

- **Formula**:

  ```
  dF = Qs * a * (1 / cosh(a * (Volt - dir * Ec * tFE)))^2;
  ```
- **Explanation**: The derivative of `F()` is critical for accurately determining how the polarization changes in response to small changes in voltage. It is used in iterative methods to converge on the correct coercive voltage value during simulation.

### 7. `dP(Volt, dir, m, b)` - Derivative of Polarization

*From line 167-175*

This function calculates the derivative of polarization (`P`) with respect to voltage, using the previously calculated slope (`m`) and the derivative of `F()`.

- **Formula**:

  ```
  dP = dF(Volt, dir) * m;
  ```
- **Explanation**: The derivative of polarization is used in calculating the rate of change of the ferroelectric charge, which is essential for modeling the dynamic behavior of the capacitor.

### 8. `dQ(Volt, dir, m, b)` - Derivative of Total Charge

*From line 177-186*

This function calculates the derivative of the total ferroelectric charge (`Q`) with respect to the input voltage. It is crucial for the iterative solution to determine the correct coercive voltage using Newton's method.

- **Formula**:

  ```
  dQ = dP(Volt, dir, m, b) + epsFE_r * EPS0 / tFE;
  ```
- **Explanation**: The derivative of the total charge is needed for numerical convergence during simulations. It provides insight into how the total charge changes with small changes in the applied voltage, which is especially important when solving for equilibrium points.

## Simulation Flow

The simulation starts by initializing key variables and determining the coercive voltage (`Vc`) and permittivity. The code then iteratively calculates the polarization and charge based on input voltages, updating the history of the system to capture the non-local memory effects typical of ferroelectric materials.

The iterative process involves finding the voltage that corresponds to a given charge using Newton's method, which is facilitated by the derivative functions discussed above. The model continuously updates its internal history, ensuring that the switching behavior accurately reflects the real-world physics of ferroelectric materials.

## References to Literature

- The use of a hyperbolic tangent to model the ferroelectric hysteresis loop is inspired by the work of Bo Jiang et al. (1997), which provides a computationally efficient way to represent the non-linear switching characteristics.
- The iterative approach for determining coercive voltage and the inclusion of both linear and non-linear components of the charge follows the methodology presented by K. Ni et al. (2018).

## Conclusion

The PFECAP model presented here provides a detailed and computationally efficient representation of ferroelectric capacitors, suitable for use in circuit simulations. By employing the Preisach model and building on existing literature, this implementation effectively captures both the non-linear switching behavior and the history-dependent nature of ferroelectric materials. The various analog functions described above work together to ensure that the simulated capacitor closely matches the expected behavior observed in experimental data.
