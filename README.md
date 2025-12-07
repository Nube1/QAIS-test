# Q-AIS: Physics-Informed Anomaly Detection for Nuclear Material Accountancy

## Multi-Modal Sensor Fusion for Nuclear Fuel Diversion Detection

## Overview

This project simulates a **Digital Twin** of a nuclear dry storage cask and implements a **Quantum-Enhanced Artificial Intelligence System (Q-AIS)** based on a physics-informed state estimator (a tuned Kalman Filter).

The goal is to demonstrate how fusing data from multiple, low-precision sensors (thermal, neutron flux, and magnetometry) can instantaneously detect clandestine, "salami-slicing" theft or diversion of nuclear material—a state variable that is otherwise difficult to measure directly.

The estimator is deliberately tuned with "stiff" process noise to encode the physical law: **Mass cannot spontaneously vanish.** This tuning forces measurement residuals to explode immediately when mass is removed, providing high-speed anomaly detection.

## 🚀 Key Features

*   **Nuclear Digital Twin (`NuclearDigitalTwin`):** Simulates the thermal and decay physics of a dry storage cask, including natural decay and heat generation.
*   **Tri-Modal Sensor Layer (`SensorGrid`):** Generates noisy, multi-modal sensor data for **Temperature**, **Neutron Flux**, and **Magnetic Field**, incorporating realistic noise profiles and decay factors.
*   **Physics-Informed State Estimation (`QAIS_Estimator`):**
    *   Implements a Kalman Filter to track the hidden states of **Temperature** and **Fuel Mass**.
    *   Uses a highly constrained **process noise matrix (`Q_cov`)** to enforce physical invariants (mass conservation).
*   **Multi-Physics Sensor Fusion:** Combines all three sensor readings to achieve a higher-fidelity estimate of the hidden state (Mass).
*   **High-Speed Anomaly Detection:** Calculates the **Mahalanobis Distance** to quantify the divergence between the physics-based prediction and the sensor measurement, indicating a high-confidence threat.
*   **CUSUM Integration:** Applies a Cumulative Sum (CUSUM) on the anomaly score to reduce false positives and reliably cross a statistically determined threshold.

## 🛠 Prerequisites

This project requires standard scientific Python libraries.

```bash
pip install numpy matplotlib scipy
```

## 📂 Code Architecture

The simulation is broken down into four distinct, coupled layers:

1.  **`Config`:** Stores all physical constants (capacitance, heat loss, decay constant) and sensor noise standard deviations.
2.  **`NuclearDigitalTwin` (The Truth):** Evolves the true state (`[Temperature, Mass]`) based on the non-linear physics equations over time.
3.  **`SensorGrid` (The Observer):** Takes the true state and applies the sensor model (mapping T/M to T/Phi/B) plus realistic Gaussian noise to generate the data that the AI receives.
4.  **`QAIS_Estimator` (The Cognitive Layer):**
    *   **Predict:** Uses the known physics model to predict the next state.
    *   **Update:** Compares the prediction to the noisy sensor measurements.
    *   **Anomaly Tuning:** The process noise for Mass (`Q_cov[1,1]`) is set extremely low (`1e-12`), encoding the AI's "belief" that Mass cannot change without an external, un-modeled force. This is the key to instant detection.

## 🚀 Usage

Run the main script to start the simulation and analysis:

```bash
python main_qais_sim.py
```
*(Note: Rename your script file to `main_qais_sim.py` for clarity.)*

### Scenario

The default scenario includes a simulated **Theft Event** at `t = 100.0` hours, where **20% of the fuel mass is instantaneously removed**.

## 📊 Expected Output & Visualization

The script prints a detailed detection report and generates a plot with four panels illustrating the system's operation.

### Certification Report Example

```
==================================================
      Q-AIS CERTIFICATION REPORT
==================================================
STATUS:        [ THREAT NEUTRALIZED ]
Attack Type:   Salami Slicing (20% Mass Diversion)
Attack Time:   t = 100.0 h
Detected At:   t = 100.25 h
Latency:       0.25 hours
Max Score:     25.3 (Threshold: 10.0)
Physics Used:  Thermodynamics + Neutronics + Magnetics
```

### Plot Panels

| Panel | Description | Key Insight |
| :--- | :--- | :--- |
| **1. Thermal Physics** | Tracks True vs. Estimated Temperature. | Thermal changes are slow and laggy, showing the limits of single-sensor detection. |
| **2. Hidden State Tracking** | Tracks True vs. Estimated Mass (the hidden state). | Shows the instantaneous drop in the True Mass and the AI's estimate struggling to keep up, but immediately deviating from the 'constant mass' prediction. |
| **3. Reference** | Shows the expected natural decay of the neutron flux. | Context for the sensor inputs, which decay over time even without theft. |
| **4. Cognitive Layer** | The **Anomaly Score** (CUSUM of Mahalanobis Distance) plotted on a log scale. | The score immediately *explodes* far past the `Detection Threshold` at the moment of the theft, proving the high-speed capability of the physics-informed fusion engine. |
