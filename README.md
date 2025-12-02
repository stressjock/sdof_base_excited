## SDOF Base-Excited Harmonic Simulator: Program README

This document explains the core logic, calculations, and visualizations of the SDOF (Single-Degree-of-Freedom) Base-Excited Harmonic Simulator, excluding the Fourier Spectrum analysis.

***

### 1. General Overview

This program models a simple mechanical system—a **mass (m)**, a **spring (k)**, and a **damper (c)**—subject to a harmonic (sine wave) motion at its **base (y)**.

It uses two primary methods:
1.  **Numerical Integration (RK4):** Solves the system's motion over time, including the transient start-up phase.
2.  **Analytical Solution:** Calculates the ideal, long-term steady-state response.

All system inputs are converted internally to **SI (Metric) units** for calculation and then converted back to the user's chosen units (Metric or English) for display.

***

### 2. Core Physics and Theory

The simulation is based on the system's **Equation of Motion (EOM)**, which governs the mass's absolute displacement ($x$):

**Equation of Motion (Absolute Displacement x):**

m \* d²x/dt² + c \* dx/dt + k \* x = c \* dy/dt + k \* y

* **m:** Mass
* **c:** Damping Coefficient
* **k:** Stiffness
* **x:** Absolute displacement of the mass
* **y:** Displacement of the base (excitation)

The RK4 engine internally solves the equation using the **relative coordinate** ($x_r = x - y$) for stability, then converts the result back to the absolute displacement $x$ for the charts.

***

### 3. Inputs and System Calculations

The user controls the following parameters:

| Category | Input Parameters |
| :--- | :--- |
| **System** | Mass (m), Stiffness (k), Damping (c) |
| **Excitation** | Base Amplitude (Y), Base Frequency (omega or f) |
| **Timing** | Total Simulation Time (s) |
| **Initial Conditions** | Initial Displacement [x(0)], Initial Velocity [v(0)] |

#### Calculated Natural Properties (Scalars)

These values are calculated based on your $m$, $k$, and $c$ inputs:

* **Natural Frequency (omega\_n):** The frequency at which the system would oscillate if given a single tap. Calculated as: $ \text{sqrt}(k/m)$ (in rad/s).
* **Natural Frequency (f\_n):** Same as above, but in Hertz (cycles per second): $ \text{omega\_n} / (2 \cdot \text{pi})$.
* **Damping Ratio (zeta):** A dimensionless value determining how quickly the oscillation fades. Calculated as: $ c / (2 \cdot \text{sqrt}(k \cdot m))$.

***

### 4. Simulation Methods

#### A. Total Response (RK4 Integration)

The `integrateRK4` function uses the highly accurate **Runge-Kutta 4th Order (RK4)** numerical method.

* **Output:** It generates a time series for the **Total Displacement** ($x$), **Total Velocity** ($v$), and **Total Acceleration** ($a$) of the mass.
* **Key Feature:** This result includes the impact of the user's **Initial Conditions** ($x(0)$ and $v(0)$) and shows the oscillation settling from the transient phase into the steady-state phase.

#### B. Steady-State Response (Analytical)

The `computeSteadyState` function calculates the ideal, long-term response of the system for a given excitation frequency, ignoring initial conditions.

* **Transmissibility Ratio (TR):** The key result is the **TR** ($|X/Y|$), which is the ratio of the mass's displacement amplitude ($X$) to the base's displacement amplitude ($Y$).
* **Steady-State Displacement:** This is a perfect sine wave with an amplitude of $Y \cdot \text{TR}$ and a fixed phase shift relative to the base motion.

***

### 5. Visualizations

The program displays three main charts:

#### A. Time History Chart

* **Plots:** Total Displacement (RK4), Steady-State Displacement (Analytical), Total Velocity, and Total Acceleration.
* **Purpose:** Shows how the motion changes over time. You can see the RK4 line (Total) starting at your initial conditions and converging onto the Analytical (Steady-State) line as the simulation progresses.

#### B. Phase Plot Chart

* **Plots:** Total Velocity ($v$) on the vertical axis vs. Total Displacement ($x$) on the horizontal axis.
* **Purpose:** Visualizes the state of the system.
    * During the **transient phase**, the curve will spiral (either inward or outward).
    * In the **steady-state phase**, the curve settles into a clean, repeating ellipse, demonstrating stable motion.

#### C. Frequency Response Chart

* **Plots:** Transmissibility (TR) and Phase Angle ($\phi$).
* **Purpose:** Shows the system's inherent behavior over a wide range of possible excitation frequencies (represented by the **Frequency Ratio $r = f / f_n$**).
    * **TR Curve:** Shows where the maximum amplification occurs (the peak near $r=1$ for low damping).
    * **Phase Curve:** Shows the delay between the base motion and the mass motion, typically shifting from 0 degrees (below resonance) to 180 degrees (far above resonance).

The Fourier Spectrum plot is a crucial tool in dynamics because it converts the system's motion from the Time Domain (the wobbly line on the time history chart) into the Frequency Domain (showing the specific sine waves that make up that wobbly line).

1. What the Axes Represent

X-Axis (Frequency in Hz): This shows all the different harmonic (sine wave) frequencies that exist in the system's motion.
It uses a Logarithmic Scale (Log Freq), which is why the bars are spread out more on the left (low frequencies) and compressed on the right (high frequencies). This is necessary to clearly see both the low natural frequency and the higher forcing frequency.

Y-Axis (Amplitude): This shows the peak amplitude of the displacement for each frequency component.

A tall bar means that frequency contributes significantly to the overall back-and-forth movement of the mass.

2. The Main Peaks You Will See

When a single-degree-of-freedom (SDOF) system is excited by a single sine wave (harmonic base excitation), the spectrum typically shows two main peaks:

A. The Primary Peak: Forcing Frequency (f)Location: This peak occurs exactly at the frequency you set for the base excitation (the input Freq).Meaning: This represents the Steady-State Response of the system. It is the dominant, permanent motion that the mass settles into after the initial starting transients have died out.

Amplitude: The height of this bar is the mass's final peak amplitude, which is determined by the base amplitude (Y) multiplied by the Transmissibility Ratio (TR) at that frequency ratio (f / f_n).

B. The Secondary Peak: Natural Frequency (f_n)Location: This peak occurs at the system's calculated Natural Frequency (f_n), which is dependent on the mass (m) and stiffness (k).Meaning: This represents the Transient Response or the system's own tendency to oscillate (free vibration).Theoretically, the transient response should decay to zero due to damping (c).However, in reality and in numerical simulations, a small residual component of the natural frequency often persists in the data, appearing as a small, secondary peak. This is because damping ($\zeta$) might be low, or because the numerical integration method introduces small, continuous "errors" that excite the system's inherent natural frequency.

3. Why "Last Cycles"?

The analysis is specifically run on the displacement data from the end of the simulation (the Last Cycles).This is done to ensure the spectrum primarily reflects the Steady-State Response.
If we analyzed the entire time history, the large, non-repeating motion at the very start (the Transient Response) would severely contaminate the spectrum, masking the important steady-state information.
By waiting until the motion settles down, we capture the most representative frequency content for the long-term, stable vibration of the system.


NOTE:

The Response of an SDOF SystemThe total motion of any SDOF system is governed by the principle of superposition:

Total Response = Transient Response + Steady-State Response

1. The Steady-State Response (Forced)
This is the motion that persists forever and is caused solely by the external force or excitation. It vibrates at the Forcing Frequency (omega).
If Base Amplitude Y > 0: The system vibrates at the Forcing Frequency omega.
If Base Amplitude Y = 0: The excitation is zero. Therefore, the Steady-State Response is zero.

2. The Transient Response (Free)This is the motion that occurs immediately after a disturbance (like non-zero initial conditions x(0) or v(0). 
It dies out over time due to damping.

1 This motion is called free vibration.
The frequency of free vibration for an underdamped system is the Damped Natural Frequency ($\omega_d$).

This response is always present if you start with non-zero initial conditions, regardless of the forcing function

Conclusion for $Y=0$

When the base amplitude $Y=0$, the system is only undergoing free vibration.
$$\text{Total Response} = \text{Transient Response} + \text{Zero}$$

Therefore, the displacement vs. time curve (the Total Response) will be:
If the system is underdamped ($\zeta < 1$): A decaying sine wave oscillating at the Damped Natural Frequency ($\omega_d$) until the motion stops.
If the system is critically damped ($\zeta = 1$) or overdamped ($\zeta > 1$): An exponentially decaying curve that returns to zero without oscillating (meaning $\omega_d = 0$).

The Undamped Natural Frequency ($\omega_n$) is only the vibration rate when both the forcing function is zero and the damping is zero ($\zeta=0$).
