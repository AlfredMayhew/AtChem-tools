# AtChem-tools
This repository brings together several tools designed to be used alongside [AtChem2](https://github.com/AtChem/AtChem2) with two main aims: 
-	To assist in the running of models through automated scripts
-	To assist in the analysis of output from AtChem2 models

Note that these tools provide greater flexibility than the tools provided as part of the AtChem2 source code, but are not designed to be used ‘out-of-the-box’. AtChemTools currently exists as a collection of Python functions that can be imported into your own custom Python scripts.

AtChemTools is intended to act as a collaborative hub for all AtChem2 users to share their tools for use alongside AtChem2. If you are an AtChem2 user, you are encouraged to **consider contributing to this repository** by suggesting additions and modifications to the existing code. If you use AtChem2 and analyse your data in a programming language other than Python, then you should still feel welcome to contribute. Get in touch and we will discuss how best to adapt the repository layout to include your contributions.

## Directory Structure

-	`AtChemTools` contains the Python functions intended to be imported into user’s own Python scripts
-	`Examples` contains a series of Python scripts intended to demonstrate common use-cases for the functions defined in the `AtChemTools` directory. Note that these scripts will not work until AtChemTools has been properly added to your Python path (as described in the Installation section).

## Installation

AtChem-tools can be installed by cloning the github repository and then adding the directory to `PYTHONPATH`. An example is given below, however you may need to alter the below commands to account for different directory structures:

```
cd ~
git clone https://github.com/AtChem/AtChem-tools/
export PYTHONPATH=$PYTHONPATH:$HOME/AtChem-tools/
```
This will allow you to import AtChem-tools in your python scripts using `import AtChemTools` or  (for example) `from AtChemTools.read_output import rate_df`.

### Package Dependencies

As well as several packages included in Python’s standard library, AtChemTools requires the import of the following packages:
-	[Pandas](https://pandas.pydata.org)
-	[Numpy](https://numpy.org)
-	[Pysolar](https://pysolar.readthedocs.io/en/latest/#)

## Building and Running Simulations
Automating the building and running of models can allow for the successive (or simultaneous) running of multiple simulations with shared behaviour. For example, you may wish to run several simulations with the same initial species concentrations except for initial VOC concentrations which vary between each simulation. By using AtChemTools' automated model running, these simulations can be build and run, with the output being saved into pandas dataframes which can then be further processed, or saved to csv files.

The following rules apply to inputs to AtChemTools functions:
- Inputs are generally provided as the following types:
    - string, integer, or float = used for individual values such as paths and values.
    - lists = used for 1-D inputs without an index such as lists of species
    - pandas series = used for 1-D inputs where there is an important index (usually species names) such as initial species concentrations
    - pandas dataframes = used for 2-D inputs where there are two important indices (usually species names and times) such as species consraints.
- All inputs should be in the same units expected by AtChem² (concentration in units of molecules cm<sup>-3</sup>, temperature in units of K, pressure in units of mbar, etc.)

Below is a description of a selection of functions defined within `AtChemTools/build_and_run.py` which can be imported into your python script using `from AtChemTools import build_and_run`, provided you have properly exported AtChemTools to PYTHONPATH.

### AtChemTools.build_and_run.write_config
Alters the configuration files for a given AtChem2 run directory based on the function arguments. Will also produce the corresponding constraint files (if needed) in the required locations.
- `atchem2_path` (str): The path to an AtChem2 directory for which the configuration files will be edited. This path should be to the root AtChem2 directory.
- `initial_concs` (pd.Series = pd.Series()): A series of initial species concentrations. The index should be a sequence of species names present in the mechanism used, and values should be the corresponding starting concentrations for each species.
- `spec_constrain` (pd.DataFrame = pd.DataFrame()): A dataframe of species constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each species that should be constrained, and the values correspond to the desired species concentration at each time. NaN can be used and will be removed for each species constraint prior to editing the configuration files.
- `spec_constant` (pd.Series = pd.Series()): A series of species that should be held at constant concentration throughout the simulation. The index should be a sequence of species names present in the mechanism used, and values should be the corresponding constant concentrations for each species.
- `env_constrain` (pd.DataFrame = pd.DataFrame()):  A dataframe of environmental constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each environmental parameter that should be constrained, and the values correspond to the desired value at each time. NaN can be used and will be removed for each environmental constraint prior to editing the configuration files.
- `photo_constant` (pd.Series = pd.Series()): A series of photolysis rates that should be held constant throughout the simulation. The index should be a sequence of J-value names (e.g. J1, J2, etc.), and values should be the corresponding constant rate for each J-value.
- `photo_constrain` (pd.DataFame = pd.DataFrame()): A dataframe of photolysis constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each J-value that should be constrained (e.g. J1, J2, etc.), and the values correspond to the desired value at each time. NaN can be used and will be removed for each photolysis constraint prior to editing the configuration files.
- `env_vals` (pd.Series = pd.Series(["298.15", "1013.25", "NOTUSED","3.91e+17", "0.41", "NOTUSED", "NOTUSED", "NOTUSED", "OPEN",  "NOTUSED"],  index = ["TEMP", "PRESS",  "RH", "H2O", "DEC", "BLHEIGHT", "DILUTE", "JFAC", "ROOF", "ASA"])): A series of environmental parameters that should be used in the simulation. The index should be a sequence of environmental parameter names, and values should be the corresponding value for each parameter. If any required environmental parameters are missing from the series, then they will be filled with default values. Will also accept non-default parameter names for custom AtChem2 versions that include additional environmental parameters.
- `spec_output` (list = []): A list of species names corresponding to desired concentration output species.
- `rate_output` (list=[]): A list of species names corresponding to desired rate output species.
### AtChemTools.build_and_run.write_model_params

### AtChemTools.build_and_run.build_model

### AtChemTools.build_and_run.run_model

### AtChemTools.build_and_run.write_build_run



## Reading Model Output
!!!
## Plotting Model Output
!!!


