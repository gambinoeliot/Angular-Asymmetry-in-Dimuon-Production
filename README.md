# Dimuon Event Simulation and Parameter Estimation at LEP

This repository contains the code and analysis for a Monte Carlo simulation of dimuon events in a collider experiment, modelled on the LEP (Large Electron-Positron Collider) at CERN. The project implements and compares four different methods for estimating the parameters of the differential cross-section for the process $e^{+}e^{-} \to \mu^{+}\mu^{-}$.

## Theoretical Background

The differential cross-section for dimuon production is given by:

$$
\frac{d\sigma}{d(\cos\theta)} \propto 1 + a_1 \cos\theta + a_2 \cos^2\theta \tag{2}
$$

where $\theta$ is the scattering angle of the muons relative to the beam axis. The $a_1 \cos\theta$ term arises from quantum interference between the electromagnetic ($\gamma$) and weak ($Z^0$) interaction channels, breaking forward-backward symmetry.

### Forward-Backward Asymmetry

The forward-backward asymmetry is defined as:

$$
A_{FB} = \frac{N_{+} - N_{-}}{N_{+} + N_{-}} \tag{3}
$$

where $N_{+}$ and $N_{-}$ are the number of events with $\cos\theta > 0$ and $\cos\theta < 0$ respectively. This relates to the cross-section parameters via:

$$
a_1 = 2\left(\frac{1}{c} + \frac{c}{3} a_2\right)A_{FB} \tag{4}
$$

for acceptance region $|\cos\theta| \leq c$.

## Methods Implemented

Four parameter estimation techniques are implemented and compared:

### 1. Forward-Backward Asymmetry Method

The forward-backward asymmetry provides a direct estimate of $a_1$ when $a_2$ is known from the total cross-section. Events are counted in the positive and negative $\cos\theta$ regions, and $A_{FB}$ is computed via equation (3). The parameter $a_1$ is then inferred using equation (4). Uncertainty is quantified by repeating over $M = 500$ independent samples and taking the standard deviation of the $a_1$ estimates.

### 2. Quadratic Regression (Gradient Descent)

A simple machine learning approach using gradient descent to minimise mean-squared error:

$$
\alpha_t = \alpha_{t-1} - \eta \frac{\partial L}{\partial\alpha}\bigg|_{\alpha = \alpha_{t-1}} \tag{5}
$$

where $\eta = 0.01$ is the learning rate, $n_{\text{iter}} = 10^4$ iterations, and initial coefficients are drawn uniformly from $[-1, 1]$. The loss function used is mean-squared error (MSE).

### 3. SciPy Curve Fitting

The `curve_fit` function from SciPy applies a non-linear least squares approach to optimise parameters by minimising the sum of squared residuals between the model and binned histogram data. It returns optimised coefficients and a covariance matrix, from which parameter uncertainties are derived as the square-rooted diagonal entries. Parameter bounds are set to $[-1, 1]$ for both $a_1$ and $a_2$.

### 4. $\chi^2$ Minimisation

A binned $\chi^2$ approach compares observed bin counts $O_i$ to expected counts $E_i$:

$$
\chi^2 = \sum_{i}^{\text{bins}} \frac{(O_i - E_i)^2}{E_i} \tag{6}
$$

Expected counts for bin $i$ are computed as:

$$
E_i = \frac{f(x_i; a_1, a_2) \times \text{Bin Width} \times N_{\text{events}}}{\text{Integral Area}} \tag{7}
$$

A 2D grid search over $a_1, a_2 \in [-1, 1]$ locates the $\chi^2$ minimum. Parameter uncertainties are extracted from the contour where $\chi^2 = \chi^2_{\text{min}} + 2$, corresponding approximately to the 68% confidence region for two parameters.

## Detector Simulation

Two detector configurations are implemented:

- **Ideal detector**: Perfect acceptance with $|\cos\theta| \leq 1.0$ and perfect angular resolution
- **Realistic detector**: Restricted acceptance with $|\cos\theta| \leq 0.9$ and angular resolution of 3 mrad, modelled by adding Gaussian noise $\theta_{\text{noise}} \sim \mathcal{N}(0, 3 \times 10^{-3})$

## Results

### Parameter Estimation Comparison ($N = 10^4$ events)

True parameters: $a_1 \approx 0.526$, $a_2 \approx 0.640$

| Method | Estimated $a_1$ | Absolute $a_1$ Uncertainty | $a_1^{\text{true}} - a_1^{\text{est}}$ | Estimated $a_2$ | Absolute $a_2$ Uncertainty | $a_2^{\text{true}} - a_2^{\text{est}}$ |
|--------|-------------------|------------------------------|------------------------------------------|-------------------|------------------------------|------------------------------------------|
| Ideal F-B | 0.5276 | 0.0244 | -0.0011 | — | — | — |
| Smeared F-B | 0.5267 | 0.0251 | -0.0002 | — | — | — |
| Quad-Reg | 0.5372 | — | -0.0107 | 0.6234 | — | 0.0164 |
| CurveFit | 0.5372 | 0.0228 | -0.0107 | 0.6234 | 0.0294 | 0.0164 |
| Chi-Squared | 0.5546 | [-0.0353, 0.0368] | -0.0281 | 0.6682 | [-0.0811, 0.0838] | -0.0284 |

### LEP Benchmark Test

For the LEP benchmark ($a_1 = 0.04$, $a_2 = 1.0$):

- Ideal detector: $a_1^{\text{idealLEP}} = 0.0445 \pm 0.0035$
- Smeared detector: $a_1^{\text{smearedLEP}} = 0.0471 \pm 0.0037$

### Key Findings

1. **Forward-backward asymmetry** proved robust even with detector smearing, with uncertainty increasing only marginally at the 3 mrad level considered.

2. **Parameter uncertainties scale as an inverse power law** with event count, demonstrating that increased statistics remain the most straightforward way to improve precision.

3. The **$\chi^2$ method** achieved near-perfect coverage (67% against theoretical 68%) with the contour set to $\chi^2_{\text{min}} + 2$, correctly containing true parameters within $1\sigma$ uncertainty.

4. For **small samples** ($N < 10^3$), error estimates become erratic, revealing the need for sufficient Monte Carlo statistics before reliable uncertainties can be obtained.

5. The **quadratic regression model** converges reliably but lacks intrinsic uncertainty estimates, limiting its use for parameter extraction.

## Dependencies

- NumPy
- SciPy (specifically `scipy.optimize.curve_fit`)
- Matplotlib (for visualisation)

## References

[1] CERN. The Large Electron-Positron Collider.  
[2] R.W. Assmann. LEP operation and performance with electron-positron collisions at 209 GeV. 2001.  
[3] Cambridge Department of Physics. Scattering theory.  
[4] LibreTexts. 16.3: The Differential Cross Section.  
[5] Eric Jones, Travis Oliphant, Pearu Peterson, et al. SciPy: Open source scientific tools for Python.  
[6] Caltech. The Minimum Chi-Square Method.  
[7] William G. Cochran. The $\chi^2$ test of goodness of fit. The Annals of Mathematical Statistics, 1952.     
[8] E. Fernandez, "Physics at LEP-1 and LEP-2", Proceedings of the Silafae-III Symposium, Cartagena de Indias, Colombia, April 2000. Available at: https://cds.cern.ch/record/850586
