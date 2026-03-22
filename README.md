# Kerr Greybody Factors from the Sasaki–Nakamura Equation

This repository computes spin-weighted Kerr greybody factors by solving the radial Sasaki–Nakamura (SN) equation in tortoise coordinates and extracting asymptotic scattering amplitudes numerically.

The main target is the gravitational case $$s=-2$$, with emphasis on extremal and near-extremal Kerr, though the same infrastructure can be adapted to lower spins.

## Overview

For Kerr perturbations, the raw Teukolsky radial equation is long-ranged,
which makes direct numerical amplitude extraction delicate.
The Sasaki–Nakamura transformation converts the problem into a short-ranged radial equation for a new variable $$X(r)$$,
whose asymptotic behavior is simple plane-wave scattering.

In the conventions used here, the SN field satisfies

$$ \frac{d^2 X}{dr_*^2} - F(r)\frac{dX}{dr_*} - U(r) X = 0, $$

or, after removal of the first-derivative term,

$$\frac{d^2 \Psi}{dr_*^2} + \left(\omega^2 - V_{\mathrm{eff}}(r_*)\right)\Psi = 0$$

This makes it possible to:

- impose a purely ingoing horizon boundary condition,
- integrate numerically to large radius,
- fit the asymptotic solution to incoming/outgoing plane waves,
- convert the fitted SN amplitudes to physical Teukolsky amplitudes,
- and finally compute the gravitational greybody factor.

## Physics setup

We work in Kerr with mass $$M$$, spin $$a=M$$, frequency $$\omega$$, and azimuthal number $$m$$.
The tortoise coordinate is defined by

$$ \frac{dr_*}{dr} = \frac{r^2+a^2}{\Delta}, \qquad \Delta = r^2 - 2Mr + a^2.  $$

At the horizon, the relevant wave number is

$$ k = \omega - m \Omega_H, \qquad \Omega_H = \frac{a}{2Mr_+},$$ with $$r_+ = M$$. 

For the horizon-normalized SN solutio $$X \sim e^{-ik r_*} \qquad (r_* \to -\infty)$$
and at infinity

$$ X \sim A^{SN}_{out} e^{+i\omega r_*} + A^{SN}_{ in} e^{-i\omega r_*}.  $$

For spin $$s=-2$$, the fitted SN amplitudes are not yet the physical amplitudes.
They must be converted to Teukolsky amplitudes before forming flux ratios.

In the conventions used here,

$$ B^{\rm inc} = -\frac{A^{SN}_{\rm in}}{4\omega^2}, \qquad B^{\rm hole} = \frac{A^{SN}_{\rm trans}}{d_{\ell m\omega}}.$$

For a horizon-normalized integration, $$A^{SN}_{\rm trans}=1$$, so the greybody factor is

$$ \Gamma_{\ell m\omega}^{(-2)} = \alpha_{\ell m\omega} \left| \frac{4\omega^2}{d_{\ell m\omega} A^{ SN}_{\rm in}} \right|^2.  $$

This is the main quantity computed by the code.

## What this repository does

- builds the SN effective potential for Kerr,
- integrates the radial equation numerically in $$r_*$$,
- extracts asymptotic amplitudes by fitting in the far region,
- computes gravitational greybody factors for fixed $$\ell,m,\omega$$,
- supports extremal Kerr $$a=M$$ and near-extremal regimes,
- provides a basis for comparison with semi-analytic MST calculations.

## Repository structure

A typical workflow is:

- `potential/`
  Construction of $$F(r)$$, $$U(r)$$, and/or the Schrödinger-form $$V_{\rm eff}(r_*)$$.

- `radial/`
  Numerical integration of the SN equation with ingoing horizon boundary conditions.

- `fit/`
  Asymptotic extraction of $$A^{ SN}_{ in}$$ and $$A^{ SN}_{out}$$

- `flux/`
  Conversion from SN amplitudes to Teukolsky amplitudes and greybody factors.

- `notebooks/`
  Mathematica notebooks for exploratory calculations and validation plots.

You can rename these sections to match the actual layout of the repo.

## Numerical method

### 1. Horizon boundary condition

At large negative tortoise coordinate,

$$X(r_*) \sim e^{-ik r_*},$$

so the numerical integration is initialized with

$$ X(r_{*\min}) = e^{-ik r_{*\min}}, \qquad X'(r_{*\min}) = -ik\, e^{-ik r_{*\min}}.  $$

### 2. Outward integration

The radial equation is integrated from the near-horizon region to a large positive tortoise coordinate \(r_{*,\max}\), chosen so that the solution is in the asymptotic far zone.

### 3. Asymptotic fit

In the far region the SN field is fit to

$$ X(r_*) \approx A^{SN}_{ out} e^{+i\omega r_*} + A^{SN}_{in} e^{-i\omega r_*}.  $$

### 4. Flux conversion

For $$s=-2$$, the fitted incoming SN amplitude is converted to the physical Teukolsky incoming amplitude, and the greybody factor is computed from the corresponding flux ratio.

## Extremal Kerr

In the extremal limit \(a=M\),

$$ r_+ = M, \qquad \Omega_H = \frac{1}{2M}, \qquad k = \omega - \frac{m}{2M}.  $$

The tortoise coordinate becomes

$$r_* = r + 2M \log(r-M) - \frac{2M^2}{r-M} +\mathrm{const.} $$

This regime is particularly delicate numerically because of the long throat near the horizon and the onset of superradiant behavior when $$k<0$$.

## Usage

A typical computation consists of:

1. choosing $$(\ell,m,\omega)$$,
2. constructing the angular eigenvalue $$\lambda_{\ell m\omega}$$,
3. building the SN potential,
4. integrating the horizon-normalized solution,
5. fitting the asymptotic amplitudes,
6. evaluating $$\Gamma_{\ell m\omega}^{(-2)}$$

In Mathematica-style pseudocode:

```Mathematica
lam = lambda[l, m, omega, a, M];
sol = solveSNIngoing[l, m, omega, a, M, lam];
ain = fitIncomingAmplitude[sol, omega];
gamma = greybodyFromSN[ain, l, m, omega, a, M, lam];
