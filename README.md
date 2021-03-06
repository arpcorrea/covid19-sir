# CovsirPhy: COVID-19 data with SIR model [![GitHub license](https://img.shields.io/github/license/lisphilar/covid19-sir)](https://github.com/lisphilar/covid19-sir/blob/master/LICENSE)[![Python version](https://img.shields.io/badge/Python-3.7|3.8-green.svg)](https://www.python.org/)

## Introduction
CovsirPhy is a Python package for COVID-19 (Coronavirus disease 2019) data analysis with SIR-derived models. Please refer to "Method" part of [COVID-19 data with SIR model](https://www.kaggle.com/lisphilar/covid-19-data-with-sir-model) notebook in Kaggle to understand the methods of analysis.

With CovsirPhy, we can apply epidemic models to COVID-19 data. Epidemic models include simple SIR and SIR-F model. SIR-F is a customized SIR-derived ODE model. To evaluate the effect of measures, parameter estimation of SIR-F will be applied to subsets of time series data in each country. Parameter change points will be determined by S-R trend analysis.

## Functionalities
- Data cleaning
    - Epidemic data: raw data must include date, country, (province), the number of confirmed/fatal/recovered cases
    - Population data: raw data must include country, (province), values of population
- Data visualization with Matplotlib
- S-R Trend analysis with Optuna and scipy.optimize.curve_fit
- Numerical simulation of ODE models with scipy.integrate.solve_ivp
- Description of ODE models
    - Basic class of ODE models
    - SIR, SIR-D, SIR-F, SIR-FV and SEWIR-F model
- Parameter Estimation of ODE models with Optuna and numerical simulation
- Simulate the number of cases with user-defined parameter values

## Inspiration
- Monitor the spread of COVID-19
- Keep track parameter values/reproductive number in each country/province
- Find the relationship of reproductive number and measures taken in each country/province

## Trying now
The author is trying to add the following functionalities.
- Speed-up S-R trend analysis and hyperparameter estimation of ODE models
- Keep track parameter values/reproductive number of all countries with a simple code
- Find relationship of reproductive number and measures automatically

If you have ideas or need new functionalities, please join this project.
Any suggestions (Github issues, pull request, comment on Kaggle notebook) are always welcomed.

## Need discussion
- Analysis with linelist of case reports
- Analysis with mobility data

## Installation
When you use this package in Kaggle notebook (need to turn on Internet option in notebook settings) or local environment with Pip,
```
# Installation
pip install --upgrade pip setuptools
pip install "git+https://github.com/lisphilar/covid19-sir#egg=covsirphy"
# Un-installation
pip install pip-autoremove
pip-autoremove covsirphy
pip uninstall pip-autoremove
```
With Pipenv environment,
```
# Installation
pipenv install "git+https://github.com/lisphilar/covid19-sir#egg=covsirphy"
# Un-installation
pipenv uninstall covsirphy
```
For developers,
```
git clone https://github.com/lisphilar/covid19-sir.git
cd covid19-sir
pip install wheel; pip install --upgrade pip; pip install pipenv
export PIPENV_VENV_IN_PROJECT=true
export PIPENV_TIMEOUT=3600
pipenv install --dev
```

## Recommended datasets
Datasets are not included in this repository, but we can download the following recommended datasets from Kaggle and GitHub. 

The necessary datasets can easily be obtained using two different: a bash script (`ìnput.sh`) or a python script (`input.py`). The bash script will not work for Windows OS. You need to setup your Kaggle account properly for both methods. Please follow the steps [provided here to ensure that you can authenticate with the Kaggle API](https://stackoverflow.com/questions/55934733/documentation-for-kaggle-api-within-python#:~:text=Here%20are%20the%20steps%20involved%20in%20using%20the%20Kaggle%20API%20from%20Python.&text=Go%20to%20your%20Kaggle%20account,json%20will%20be%20downloaded).

If you choose to use the python script, note that simply putting the `kaggle.json` in the same folder as `input.py` before executing it will allow `input.py` to find it (not necessary to modify environment variables).

Please read Bash code `input.sh` in the top directory of this repository to better understand its usage.

### Kaggle
Kaggle API key and Kaggle package are necessary. Please read [How to Use Kaggle: Public API](https://www.kaggle.com/docs/api).

#### The number of cases
Primary source: [COVID-19 Data Repository by CSSE at Johns Hopkins University](https://github.com/CSSEGISandData/COVID-19)  
Secondary source: [Novel Corona Virus 2019 Dataset by SRK](https://www.kaggle.com/sudalairajkumar/novel-corona-virus-2019-dataset)  
We can download this dataset from the primary source directly. Please refer to "Quick usage" section.

### The number of cases in Japan
Primary source: [Ministry of Health, Labour and Welfare HP (in English)](https://www.mhlw.go.jp/stf/seisakunitsuite/bunya/newpage_00032.html)  
Secondary source: [Secondary source: COVID-19 dataset in Japan by Lisphilar](https://www.kaggle.com/lisphilar/covid19-dataset-in-japan)  
We can download this dataset from the secondary source directly. Please refer to "Quick usage" section.

#### Total population
[covid19 global forecasting: locations population by Dmitry A. Grechka](https://www.kaggle.com/dgrechka/covid19-global-forecasting-locations-population)  

### GitHub
#### OxCGRT: Measures taken by each country and response scores
[Thomas Hale, Sam Webster, Anna Petherick, Toby Phillips, and Beatriz Kira. (2020).  
Oxford COVID-19 Government Response Tracker. Blavatnik School of Government.](https://github.com/OxCGRT/covid-policy-tracker)  
We can download this dataset from the primary source directly. Please refer to "Quick usage" section.



## Quick usage
Example Python codes are in `example` directory. With Pipenv environment, we can run the Python codes with Bash code `example.sh` in the top directory of this repository.

### Preparation
```Python
import covsirphy as cs
cs.__version__
```
Set the directory to save the datasets.
```Python
data_loader = cs.DataLoader("input")
```
Download JHU dataset and perform data cleaning.
```Python
jhu_data = data_loader.jhu()
# Show citation
print(jhu_data.citation)
# Return the cleaned dataset as a dataframe
jhu_data.cleaned()
```
(Optional) We can replace JHU data with country-specific dataset.
```Python
# As an example, read Japan dataset
japan_data = data_loader.japan()
print(japan_data.citation)
# Replace records of Japan with Japan-specific dataset
jhu_data.replace(japan_data)
ncov_df = jhu_data.cleaned()
```
Perform data cleaning of population dataset.
```Python
# Read population dataset
pop_data = cs.Population("input/locations_population.csv")
# We can add a new record
pop_data.update(country="Example", province="-", value=1_000_000)
# Return dictionary
pop_data.to_dict(country_level=True)
```
Perform data cleaning of OxGCRT dataset.
```Python
oxcgrt_data = data_loader.oxcgrt()
# Show citation
print(oxcgrt_data.citation)
# Data cleaning
oxcgrt_df = oxcgrt_data.cleaned()
# Create a subset for a country with ISO3 country code
jpn_oxcgrt_df = oxcgrt_data.subset(iso3="JPN")
```

### Scenario analysis
As an example, use dataset in Italy.
#### Check records
```Python
ita_scenario = cs.Scenario(jhu_data, pop_data, country="Italy", province=None)
```
See the records as a figure.
```Python
ita_record_df = ita_scenario.records()
```
#### S-R trend analysis
Show S-R trend to determine the number of change points.
```Python
ita_scenario.trend()
```
As an example, set the number of change points as 4.
```Python
ita_scenario.trend(n_points=4, set_phases=True)
```
Start/end date of the four phase were automatically determined. Let's see.
```Python
print(ita_scenario.summary())
```
#### Hyperparameter estimation of ODE models
As an example, use SIR-F model.
```Python
ita_scenario.estimate(cs.SIRF)
print(ita_scenario.summary())
```
We can check the accuracy of estimation with a figure.
```Python
# Table
ita_scenario.estimate_accuracy(phase="1st")
# Get a value
ita_scenario.get("Rt", phase="4th")
# Show parameter history as a figure
ita_scenario.param_history(targets=["Rt"], divide_by_first=False, box_plot=False)
ita_scenario.param_history(targets=["rho", "sigma"])
```
#### Prediction of the number of cases
we can add some future phases.
```Python
# if needed, clear the registered future phases
ita_scenario.clear(name="Main")
# Add future phase to main scenario
ita_scenario.add_phase(name="Main", end_date="01Aug2020")
# Get parameter value
sigma_4th = ita_scenario.get("sigma", name="Main", phase="4th")
# Add future phase with changed parameter value to new scenario
sigma_6th = sigma_4th * 2
ita_scenario.add_phase(end_date="31Dec2020", name="Medicine", sigma=sigma_6th)
ita_scenario.add_phase(days=30, name="Medicine")
print(ita_scenario.summary())
```
Then, we can predict the number of cases and get a figure.
```Python
# Prediction and show figure
sim_df = ita_scenario.simulate(name="Main")
# Describe representative values
print(ita_scenario.describe())
```

## Citation
Lisphilar, 2020, Kaggle notebook, COVID-19 data with SIR model, https://www.kaggle.com/lisphilar/covid-19-data-with-sir-model

CovsirPhy development team, 2020, GitHub repository, CovsirPhy, Python package for COVID-19 data with SIR model, https://github.com/lisphilar/covid19-sir
