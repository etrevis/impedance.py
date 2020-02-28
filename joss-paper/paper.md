---
title: 'impedance.py: A Python package for electrochemical impedance analysis'
tags:
  - Python
  - electrochemistry
  - impedance
  - lithium-ion batteries
  - fuel cells
  - corrosion
authors:
  - name: Matthew D. Murbach
    email: matt@hivebattery.com
    orcid: 0000-0002-6583-5995
    affiliation: 1
  - name: Brian Gerwe
    email: brian.s.gerwe@gmail.com
    orcid: 0000-0002-1184-8483
    affiliation: 2
  - name: Neal Dawson-Elli
    affiliation: 3
  - name: Lok-kun Tsui
    email: lktsui@unm.edu
    orcid: 0000-0001-7381-0686
    affiliation: 4
affiliations:
 - name: Hive Battery Labs
   index: 1
 - name: University of Washington, Department of Chemical Engineering
   index: 2
 - name: PayScale, Inc.
   index: 3
 - name: University of New Mexico, Center for MicroEngineered Materials
   index: 4
date: 19 February 2020
bibliography: paper.bib
---

`impedance.py` is a community-driven Python package for making the analysis of electrochemical
impedance spectroscopy (EIS) data easier and more reproducible. `impedance.py` currently provides several useful features commonly used in the typical impedance analysis workflow:

- _data ingestion_: functions for importing data from a variety of instruments and file types
- _data validation_: easy-to-use methods for checking measurement validity
- _model fitting_: a simple and powerful interface for quickly fitting models to analyze data
- _model selection_: parameter error estimates and model confidence bounds
- _visualization_: interactive and publication-ready Nyquist and Bode plots

# Background

Electrochemical impedance spectroscopy (EIS) is a powerful technique for noninvasively
probing the physicochemical processes governing complex electrochemical systems by examining the voltage-current response
 to sinusoidal perturbations [@orazem_electrochemical_2008]. EIS data interpretation is commonly approached by constructing equivalent circuit models to represent
 the system response. Researchers gain insight by attributing elements in a proposed model to processes in their system and evaluating
 how well the model fits data. This approach assumes the voltage-current response is linear and causal, and the system is stable over
 the measurement time. As such, researchers must also ensure their data complies with these assumptions – often through some implementation
 of Kramers-Kronig analysis. Following the approach outlined above, workers have applied EIS to problems ranging
 from measuring the dielectric properties of biological systems to corrosion rates of coated metals. More recently, EIS has been widely adopted
 for studying processes governing batteries and fuel cells [@orazem_electrochemical_2008].

# Statement of Need
To date, typical impedance analysis solutions have relied on either instrument-specific, proprietary software or ad hoc, lab-specific code
 written for internal use. In addition to access barriers, these solutions can be restrictive to defining custom circuit elements and processing
 large datasets. By providing an open-source, community-driven package for the full impedance analysis pipeline from data management to parameter
 extraction to publication ready figures, `impedance.py` seeks to encourage reproducible, easy-to-use, and transparent analysis. In addition to decades
 of electrochemical research, many new methods for validating [@schonleber_method_2014] and analyzing [@murbach_analysis_2018; @buteau_analysis_2019]
 impedance spectra have been developed by researchers. By lowering the barrier to use tried-and-true methods along side cutting-edge analytical
 techniques via a consistent interface, `impedance.py` serves to grow as a community repository of best-practices while facilitating the adoption
 of new techniques.

# Current `impedance.py` functionality

## Data Ingestion
Once you've collected your data, the last thing you want to do is worry about how
to turn the files generated by the instrument into results you can learn from. `impedance.py`
offers easy-to-use functions for importing data from BioLogic, Gamry, PARSTAT, VersaStudio, and ZView files. Don't see your instrument yet?
 [Create an issue with a sample file](https://github.com/ECSHackWeek/impedance.py/issues/new?assignees=&labels=&template=data-file-support-request.md&title=%5BDATA%5D)
 and help build the project!

## Data validation
Ensuring that the system under measurement is linear, stable, and causal is an important, but
often overlooked part of impedance analysis. `impedance.py` provides several methods 
(measurement models, Lin-KK algorithm [@schonleber_method_2014]) for data validation as a part of 
the same easy-to-use package.

## Equivalent circuit fitting
`impedance.py` equivalent circuit fitting combines an extremely flexible circuit definition with a range of available circuit elements --
 from simple L, R, C, and constant-phase elements to electrochemistry specific elements such as Warburg, Gerischer [@boukamp_gerischer_element_2003],
 and TLM elements [@paasch_theory_1993]. Custom elements can be easily added to a single users installation or to the overall project by 
 [creating an issue with a sample file](https://github.com/ECSHackWeek/impedance.py/issues/new?assignees=&labels=&template=data-file-support-request.md&title=%5BDATA%5D).
 You can put bounds on parameters, hold particular circuit elements constant, perform weighted fitting, and even save and load models to a human readable
 .json file. Fitting is performed by non-linear least squares regression of the circuit model to impedance data via `curve_fit` from the `scipy.optimize` package.

## Model selection
One of the biggest challenges in extracting parameters from EIS spectra is quantitatively
evaluating the fit of different models. `impedance.py` returns estimated parameter error
bars with every fit and comes with a built in method for calculating the confidence bounds
of a fit model.

## Data visualization
A key part of EIS interpretation is examining the spectra shapes, which can be distorted by improperly defined axes.
`impedance.py` eliminates the tedium of manually formatting plots by providing built-in methods to generate clean,
 publication-ready figures from interactive plots via Altair [@vanderplas_altair_2018] to flexible Nyquist or Bode plots via
matplotlib [@hunter_matplotlib_2007].


# An example of the simple model API
The documentation for `impedance.py` contains
[a guide on getting started](https://impedancepy.readthedocs.io/en/latest/getting-started.html)
and several examples of what a typical analysis workflow might look like
using the package. Here we show how importing data, defining and fitting an equivalent circuit 
model, and visualizing the results can be done with just a handful of lines in `impedance.py`:

```python
# 1. loading in data
from impedance.preprocessing import readFile
f, Z = readFile('exampleData.csv')

# 2. importing and initializing a circuit:
from impedance.models.circuits import CustomCircuit
initial_guess = [1e-8, .01, .005, .1, .9, .005, .1, .9, .1, 200]
circuit = CustomCircuit('L0-R0-p(R1,E1)-p(R2,E2)-W1',
                        initial_guess=initial_guess)

# 3. fitting the circuit to the data
circuit.fit(f, Z)
print(circuit)
```

```text
Circuit string: L0-R0-p(R1,E1)-p(R2,E2)-W1
Fit: True

Initial guesses:
     L0 = 1.00e-08 [H]
     R0 = 1.00e-02 [Ohm]
     R1 = 5.00e-03 [Ohm]
   E1_0 = 1.00e-01 [Ohm^-1 sec^a]
   E1_1 = 9.00e-01 []
     R2 = 5.00e-03 [Ohm]
   E2_0 = 1.00e-01 [Ohm^-1 sec^a]
   E2_1 = 9.00e-01 []
   W1_0 = 1.00e-01 [Ohm]
   W1_1 = 2.00e+02 [sec]

Fit parameters:
     L0 = 1.34e-07  (+/- 4.68e-08) [H]
     R0 = 1.48e-02  (+/- 8.24e-04) [Ohm]
     R1 = 7.94e-03  (+/- 2.04e-03) [Ohm]
   E1_0 = 1.15e+00  (+/- 7.75e-01) [Ohm^-1 sec^a]
   E1_1 = 6.79e-01  (+/- 1.22e-01) []
     R2 = 8.21e-03  (+/- 1.29e-03) [Ohm]
   E2_0 = 4.88e+00  (+/- 5.25e-01) [Ohm^-1 sec^a]
   E2_1 = 9.24e-01  (+/- 4.47e-02) []
   W1_0 = 1.39e-01  (+/- 8.95e-02) [Ohm]
   W1_1 = 1.26e+03  (+/- 1.60e+03) [sec]
```

```python
# 4. visualize the results
circuit.plot()
```

![Interactive impedance plots are as easy as `.plot()`!](./example.png)

# Acknowledgements

We thank participants on the 2018 Electrochemical Society (ECS) Hack Week team
in Seattle, WA as well as Dan Schwartz and David Beck for their guidance.
An up-to-date [list of contributors can be found on GitHub](https://github.com/ECSHackWeek/impedance.py#contributors-). Example data is from [@murbach_nonlinear_2018].

# References