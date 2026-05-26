\# Probabilistic Modeling, Copula Theory, and Portfolio Risk Analysis



> A rigorous, end-to-end framework for structural reliability assessment and financial portfolio risk quantification using multivariate probabilistic models, copula-based dependence structures, and Monte Carlo simulation.



\---



\## Table of Contents



\- \[Overview](#overview)

\- \[Project Structure](#project-structure)

\- \[Methodology](#methodology)

&#x20; - \[Part I — Structural Reliability \& Fragility Analysis](#part-i--structural-reliability--fragility-analysis)

&#x20; - \[Part II — Portfolio Risk \& Capital Buffer Estimation](#part-ii--portfolio-risk--capital-buffer-estimation)

\- \[Key Results](#key-results)

&#x20; - \[Structural Analysis Results](#structural-analysis-results)

&#x20; - \[Portfolio Risk Results](#portfolio-risk-results)

\- \[Technical Stack](#technical-stack)

\- \[Installation \& Usage](#installation--usage)

\- \[Mathematical Background](#mathematical-background)

\- \[Figures \& Visualizations](#figures--visualizations)

\- \[Discussion \& Insights](#discussion--insights)

\- \[Report](#report)

\- \[Author](#author)

\- \[References](#references)



\---



\## Overview



This project develops and applies a unified probabilistic framework — combining parametric marginal distribution fitting with copula-based joint dependence modeling — to two distinct but structurally analogous risk problems:



1\. \*\*Structural Reliability\*\*: Probabilistic characterization of seismic structural response for two reinforced concrete buildings subjected to 10,000 nonlinear dynamic analyses, culminating in copula-derived fragility functions.



2\. \*\*Portfolio Risk Analysis\*\*: Dependence-aware estimation of portfolio cash flow distributions and extreme-loss quantiles for a two-asset infrastructure portfolio, with direct implications for regulatory capital buffer sizing.



The central insight motivating both analyses is that \*\*marginal distributions and dependence structure are distinct objects\*\*. Two systems can be indistinguishable at the level of means and variances yet behave very differently in the tails — precisely the region that governs structural safety and financial solvency. Copula theory provides the natural language to separate and model these two components independently.



\---



\## Project Structure



```

├── notebooks/

│   ├── Problem\_1.ipynb          # Structural reliability analysis (Jupyter)

│   └── Problem\_2.ipynb          # Portfolio risk analysis (Jupyter)

├── data/

│   ├── Data\_I.mat               # Building I — 10,000 nonlinear structural analyses

│   ├── Data\_II.mat              # Building II — 10,000 nonlinear structural analyses

│   └── Data\_Prob\_2.csv          # Portfolio asset returns (N = 800 paired observations)

├── figures/                     # All generated plots (auto-populated on run)

├── report/

│   └── HW5\_Report.pdf           # Full technical report (25 pages)

└── README.md

```



\---



\## Methodology



\### Part I — Structural Reliability \& Fragility Analysis



Each dataset contains paired observations of:

\- \*\*Spectral Acceleration\*\* `Sa` \[cm/s²] — seismic intensity measure (input)

\- \*\*Engineering Demand Parameter\*\* `EDP` \[cm] — maximum inter-storey drift (structural response)



The analysis proceeds through the following pipeline:



\#### 1. Marginal Distribution Fitting



Lognormal distributions are assumed for both `Sa` and `EDP`, motivated by the strict positivity and empirical log-normality of seismic intensity and drift data. Parameters are estimated by maximum likelihood on the log-transformed data:



```

μ̂\_ln = (1/N) Σ ln(xₙ)

σ̂\_ln = sqrt( (1/(N-1)) Σ (ln(xₙ) − μ̂\_ln)² )

```



\#### 2. Probability Integral Transform (Unit Cube)



Each observation is mapped to uniform pseudo-observations via the fitted lognormal CDF:



```

v₁ = F\_Sa(sa),    v₂ = F\_EDP(edp)

```



The resulting scatter in `\[0,1]²` isolates the pure dependence structure, stripped of marginal effects.



\#### 3. Standard Normal Space \& Log-Space Diagnostics



Two equivalent diagnostic transformations are applied:

\- \*\*Probit transform\*\*: `u₁ = Φ⁻¹(v₁)`, `u₂ = Φ⁻¹(v₂)` — deviations from elliptical symmetry indicate non-Gaussian dependence.

\- \*\*Log transform\*\*: `(ln Sa, ln EDP)` — if the joint distribution were bivariate lognormal, this scatter would be an exact ellipse.



\#### 4. Copula Parameter Estimation



Two copula families are fitted:



\*\*Gaussian Copula\*\* — parameterized by `ρ₀`, estimated as the Pearson correlation of the probit-transformed pseudo-observations:

```

ρ̂₀ = corr( Φ⁻¹(F\_Sa(sa)), Φ⁻¹(F\_EDP(edp)) )

```



\*\*Clayton Copula\*\* — parameterized by `θ > 0`, estimated via Kendall's rank correlation:

```

θ̂ = 2τ̂ / (1 − τ̂)

```



\#### 5. Model Selection via Log-Likelihood



The preferred copula is selected by maximizing the copula log-likelihood:

```

log L = Σ log c( F\_Sa(saₙ), F\_EDP(edpₙ) )

```

evaluated separately for the Gaussian and Clayton densities.



\#### 6. Joint Density, Conditional CDFs \& Fragility Functions



The complete joint density is assembled via Sklar's theorem:

```

f\_{Sa,EDP}(sa, edp) = c( F\_Sa(sa), F\_EDP(edp) ) · f\_Sa(sa) · f\_EDP(edp)

```



Conditional CDFs `F(EDP | Sa = sa)` are derived analytically (Gaussian copula) or by partial differentiation (Clayton copula), and fragility functions `P(EDP > edpₖ | Sa)` are computed for four damage-state thresholds: `edp ∈ {2, 4, 6, 8}` cm.



\#### 7. Equivalence: Joint Lognormal vs. Gaussian Copula + Lognormal Margins



A theoretical and numerical verification confirms that the Gaussian copula with lognormal marginals is mathematically identical to a bivariate joint lognormal distribution. The maximum relative difference between the two formulations is verified at machine precision (`εmax ≈ 6 × 10⁻¹³`).



\---



\### Part II — Portfolio Risk \& Capital Buffer Estimation



The portfolio `Y = X₁ + X₂` consists of two infrastructure assets with gains/losses in Euros (`N = 800` paired observations).



\#### 1. Marginal Distribution Selection



Both assets exhibit negative values (\~10–12% of observations), ruling out the lognormal distribution. \*\*Normal marginals\*\* are fitted by MLE and validated visually via histogram overlay and normal QQ plots.



\#### 2. Copula Fitting \& Selection



The same Gaussian and Clayton copula framework from Part I is applied. The Clayton copula is selected based on log-likelihood (83.60 vs. 58.69), reflecting the known tendency of financial assets to co-move more strongly during adverse conditions (lower-tail dependence).



\#### 3. Monte Carlo Simulation (N\_sim = 500,000)



Two joint models are simulated:

\- \*\*Model I\*\*: Gaussian copula + normal marginals

\- \*\*Model II\*\*: Clayton copula + normal marginals (sampled via exact conditional inversion — Marshall–Olkin algorithm)



From each simulation, the following risk metrics are estimated:

\- \*\*Expected cash flow\*\*: `E\[Y] = E\[X₁] + E\[X₂]` (copula-invariant by linearity of expectation)

\- \*\*Capital buffer\*\*: `F\_Y⁻¹(10⁻³)` — the 0.1th percentile of the portfolio loss distribution



\---



\## Key Results



\### Structural Analysis Results



| Property | Building I | Building II |

|---|---|---|

| `μ̂\_ln` (Sa) | 5.392 | 5.389 |

| `σ̂\_ln` (Sa) | 0.468 | 0.470 |

| `μ̂\_ln` (EDP) | 1.294 | 1.293 |

| `σ̂\_ln` (EDP) | 0.427 | 0.429 |

| Gaussian copula `ρ̂₀` | 0.802 | 0.863 |

| Clayton copula `θ̂` | 2.913 | 4.951 |

| Kendall's `τ̂` | 0.593 | 0.712 |

| \*\*Best copula model\*\* | \*\*Gaussian\*\* | \*\*Clayton\*\* |

| log L (Gaussian) | 5,147.86 | 6,827.84 |

| log L (Clayton) | 3,040.44 | 9,526.27 |



Despite nearly identical marginal distributions, Buildings I and II differ fundamentally in dependence structure. Building I exhibits elliptical (Gaussian) dependence; Building II displays pronounced upper-tail concentration, requiring the Clayton copula to prevent unconservative fragility estimates at high ground motion intensities.



\### Portfolio Risk Results



| Quantity | Model I (Gaussian) | Model II (Clayton) |

|---|---|---|

| `E\[Y]` \[EUR] | 57.18 | 57.12 |

| Theoretical `E\[Y]` \[EUR] | 57.13 | 57.13 |

| `F\_Y⁻¹(10⁻³)` \[EUR] | −57.80 | \*\*−73.26\*\* |

| Capital buffer difference | — | \*\*+15.46 EUR (+27%)\*\* |



The expected cash flow is copula-independent. The capital buffer, however, differs by \*\*27%\*\* between models. A risk manager relying solely on the Gaussian copula would face a \*\*4× higher probability of insolvency\*\* at the target risk level of `10⁻³`.



\---



\## Technical Stack



| Component | Tool |

|---|---|

| Language | Python 3.x |

| Numerical computing | NumPy, SciPy |

| Statistical modeling | SciPy (`stats`), `statsmodels` |

| Monte Carlo simulation | NumPy random number generation |

| Visualization | Matplotlib, Seaborn |

| Notebook environment | Jupyter Notebook / JupyterLab |

| Data I/O | `scipy.io` (MATLAB `.mat`), Pandas (CSV) |



\---



\## Installation \& Usage



\### 1. Clone the repository



```bash

git clone https://github.com/Oluwakorede001/copula-risk-analysis.git

cd copula-risk-analysis

```



\### 2. Create a virtual environment and install dependencies



```bash

python -m venv venv

source venv/bin/activate        # Windows: venv\\Scripts\\activate

pip install -r requirements.txt

```



A minimal `requirements.txt`:



```

numpy

scipy

pandas

matplotlib

seaborn

jupyter

```



\### 3. Run the notebooks



```bash

jupyter lab

```



Open `notebooks/Problem\_1.ipynb` for the structural analysis and `notebooks/Problem\_2.ipynb` for the portfolio risk analysis. All figures are saved automatically to the `figures/` directory.



\---



\## Mathematical Background



\### Sklar's Theorem



Any multivariate joint distribution `F(x₁, x₂)` can be decomposed as:

```

F(x₁, x₂) = C( F₁(x₁), F₂(x₂) )

```

where `C : \[0,1]² → \[0,1]` is a \*\*copula\*\* — a joint distribution on the unit square with uniform marginals. The copula uniquely encodes the dependence structure, completely separated from the marginals `F₁`, `F₂`.



\### Gaussian Copula Density



```

c^Ga(v₁, v₂; ρ₀) = (1 / √(1−ρ₀²)) · exp( −\[ρ₀²(z₁² + z₂²) − 2ρ₀ z₁ z₂] / \[2(1−ρ₀²)] )

```

where `zᵢ = Φ⁻¹(vᵢ)`. This copula has \*\*zero tail dependence\*\* — extreme events in both variables are asymptotically independent.



\### Clayton Copula Density



```

c^Cly(v₁, v₂; θ) = (1+θ)(v₁v₂)^{−(1+θ)} · (v₁^{−θ} + v₂^{−θ} − 1)^{−(1+2θ)/θ}

```

The Clayton copula has \*\*positive lower-tail dependence\*\* with coefficient `λ\_L = 2^{−1/θ}`. As `θ → ∞`, dependence becomes perfect; as `θ → 0`, the variables become independent.



\### Conditional CDF Derivation



For the Gaussian copula:

```

F(edp | sa) = Φ( \[Φ⁻¹(F\_EDP(edp)) − ρ̂₀ Φ⁻¹(F\_Sa(sa))] / √(1−ρ̂₀²) )

```



For the Clayton copula (partial derivative):

```

F(edp | sa) = v₁^{−(1+θ)} · (v₁^{−θ} + v₂^{−θ} − 1)^{−(1+θ)/θ}

```



\### Fragility Functions



```

P(EDP > edpₖ | Sa = sa) = 1 − F(edpₖ | sa)

```



Evaluated for damage thresholds `edpₖ ∈ {2, 4, 6, 8}` cm as a function of spectral acceleration `sa`.



\---



\## Figures \& Visualizations



The following outputs are generated by the notebooks:



| Figure | Description |

|---|---|

| Marginal KDE plots | Lognormal/normal marginal fits for Sa, EDP, X₁, X₂ |

| Unit cube scatter | `(v₁, v₂)` — dependence structure after marginal removal |

| Standard normal space scatter | `(u₁, u₂)` — Gaussianity diagnostic |

| Log-transform scatter | `(ln Sa, ln EDP)` — bivariate normality diagnostic |

| Joint density surfaces \& contours | 3D surface and 2D contour plots under Gaussian and Clayton copulas |

| Conditional CDFs | `F(EDP | Sa)` at four intensity levels |

| Fragility functions | `P(EDP > edpₖ | Sa)` for four damage states |

| Joint lognormal vs. Gaussian copula comparison | Numerical equivalence verification |

| Portfolio PDF | `f\_Y(y)` under both copula models with VaR markers |

| Survival function (semi-log) | `P(Y > y)` in log-scale — tail divergence visualization |



\---



\## Discussion \& Insights



\*\*1. The copula choice is invisible in the bulk, critical in the tails.\*\*

Both structural buildings and both financial assets produce nearly identical marginal distributions and similar central moments. The differences emerge exclusively in the joint tails — the region governing fragility, insolvency, and capital adequacy.



\*\*2. Tail dependence has engineering and financial consequences.\*\*

For Building II, applying the Gaussian copula (which has zero tail dependence) would underestimate co-occurrence of extreme `(Sa, EDP)` pairs, leading to unconservative fragility estimates at high intensities — a potential safety risk. For the financial portfolio, the same error translates to a 27% underestimation of the required capital buffer and a 4× increase in the probability of insolvency.



\*\*3. The joint lognormal model is a special case.\*\*

When lognormal marginals are combined with a Gaussian copula, the result is mathematically identical to a bivariate joint lognormal distribution. This project verifies this equivalence numerically to machine precision (`εmax ≈ 6 × 10⁻¹³`), confirming that the Gaussian copula framework strictly generalizes the classical joint lognormal model widely used in earthquake engineering.



\*\*4. Expected value is copula-invariant; risk measures are not.\*\*

By linearity of expectation, `E\[X₁ + X₂]` is always equal to `E\[X₁] + E\[X₂]`, regardless of dependence structure. This is confirmed in simulation (difference < 0.06 EUR across 500,000 draws). Variance, quantiles, and all risk measures beyond the mean are sensitive to the copula choice.



\*\*5. Kendall's τ as a robust dependence summary.\*\*

Kendall's rank correlation is used as the primary dependence estimator for the Clayton parameter, offering robustness to marginal misspecification that Pearson correlation does not provide. The p-value for positive dependence in the portfolio data (`p = 5.82 × 10⁻²⁴`) confirms that dependence is not a statistical artifact.



\---



\## Report



A complete 25-page technical report accompanies this project, covering:

\- Full theoretical derivations of all copula densities and conditional CDFs

\- Detailed model selection justification (heuristic + log-likelihood)

\- Numerical comparison tables for all fitted parameters

\- Extended discussion of fragility curves and their practical implications for performance-based earthquake engineering

\- Capital adequacy analysis and regulatory risk implications



📄 \[View Report (PDF)](report/HW5\_Report.pdf)



\---



\## Author



\*\*Oluwakorede Hezekiah Oyewole\*\*

MSc — Mechanics of Sustainable Materials and Structures (MS2)

University of Trento, Italy



\---



\## References



1\. Sklar, A. (1959). \*Fonctions de répartition à n dimensions et leurs marges.\* Publications de l'Institut de Statistique de l'Université de Paris, 8, 229–231.



2\. Nelsen, R. B. (2006). \*An Introduction to Copulas\* (2nd ed.). Springer.



3\. Joe, H. (2014). \*Dependence Modeling with Copulas.\* Chapman \& Hall/CRC.



4\. Frees, E. W., \& Valdez, E. A. (1998). Understanding relationships using copulas. \*North American Actuarial Journal\*, 2(1), 1–25.



5\. Cornell, C. A., Jalayer, F., Hamburger, R. O., \& Foutch, D. A. (2002). Probabilistic basis for 2000 SAC federal emergency management agency steel moment frame guidelines. \*Journal of Structural Engineering\*, 128(4), 526–533.



6\. Genest, C., \& Favre, A. C. (2007). Everything you always wanted to know about copula modeling but were afraid to ask. \*Journal of Hydrologic Engineering\*, 12(4), 347–368.



7\. Shang, H., \& Donahue, M. (2014). \*Financial Risk Management with Copulas.\* Risk Books.



8\. Baker, J. W. (2015). Efficient analytical fragility function fitting using dynamic structural analysis. \*Earthquake Spectra\*, 31(1), 579–599.



\---



\*This project was developed as part of research in probabilistic risk analysis and structural reliability at the University of Trento. The methodologies are directly applicable to performance-based earthquake engineering, insurance loss modeling, and financial risk management.\*

