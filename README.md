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
-	[pandas](https://pandas.pydata.org)
-	[NumPy](https://numpy.org)
-	[matplotlib](https://matplotlib.org/stable/)
-	[Pysolar](https://pysolar.readthedocs.io/en/latest/#)

## Building and Running Simulations
Automating the building and running of models can allow for the successive (or simultaneous) running of multiple simulations with shared behaviour. For example, you may wish to run several simulations with the same initial species concentrations except for initial VOC concentrations which vary between each simulation. By using AtChemTools' automated model running, these simulations can be build and run, with the output being saved into pandas dataframes which can then be further processed, or saved to csv files.

The following rules apply to inputs to AtChemTools functions:
- Inputs are generally provided as the following types:
    - string, integer, or float = used for individual values such as paths and values.
    - lists = used for 1-D inputs without an index such as lists of species
    - pandas series = used for 1-D inputs where there is an important index (usually species names) such as initial species concentrations
    - pandas dataframes = used for 2-D inputs where there are two important indices (usually species names and times) such as species consraints.
- All inputs should be in the same units expected by AtChem2 (concentration in units of molecules cm<sup>-3</sup>, temperature in units of K, pressure in units of mbar, etc.)

Below is a description of a selection of functions defined within `AtChemTools/build_and_run.py` which can be imported into your python script using `from AtChemTools import build_and_run`, provided you have properly exported AtChemTools to PYTHONPATH.

### AtChemTools.build_and_run.write_config
Alters the configuration files for a given AtChem2 model directory based on the function arguments. Will also produce the corresponding constraint files (if needed) in the required locations.
- `model_path` (str): The path to an AtChem2 model directory for which the configuration files will be edited. This path should be to the model directory, e.g. "AtChem2/model/".
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
Edits the `model.parameters` configuration file in the specified location based on the information provided.
- `atchem2_path` (str): The path to an AtChem2 model directory for which the `model.parameters` file will be edited. This path should be to the model directory, e.g. "AtChem2/model/".
- `nsteps` (int): The number of steps that the model should run for.
- `model_tstep` (int): The length of each timestep in seconds. This model timestep is currently also used for the concentration and rate output timestep, meaning models will output data for every step of the model.
- `tstart` (int): The start time of the model in seconds. This should be in UTC time in order to properly calculate photolysis rates.
- `day` (int): Day of the month of the model start time (used for photolysis calculations).
- `month` (int): Month of the year of the model start time (used for photolysis calculations).
- `year` (int): Year of the model start time (used for photolysis calculations).
- `lat` (float): latitude of the model location  (used for photolysis calculations).
- `lon` (float): longitude of the model location  (used for photolysis calculations). Note that, as is explained in the AtChem2 documentation, **longitude west is positive**. This is the opposite of the standard convention where east is positive.
### AtChemTools.build_and_run.build_model
Runs the AtChem2 bash script to build the model at the specified path.
- `atchem2_path` (str): The path to an AtChem2 directory which will be build. This path should be to the root AtChem2 directory.
- `mechanism_path` (str): The path to the mechanism used to build the model.
- `uselock` (bool = False): Determines whether to use a file lock when building the model to prevent errors from simultaneous builds. There is no problem using the same atchem executable for multiple simulations (provided different `model` directories are used), but trying to build two models simultaneously can cause issues. The lock can slow down runs if there are lots of runs (e.g. >100) waiting for lock release before building.
### AtChemTools.build_and_run.run_model
Runs the specified AtChem2 excecutable. The build script should be run before using this function.
- `atchem2_path` (str): The path to an AtChem2 directory which will be run. This path should be to the root AtChem2 directory.
- `model_path` (str = ""): The path to an AtChem2 model directory used as the first argument to the atchem2 executable. If this is left as the default empty string, then no model directory argument will be passed to the executable.
### AtChemTools.build_and_run.write_build_run
Uses many of the above functions to edit the configuration files in a given directory, build the model with a specified mechanism, run the model, and save the output to pandas dataframes which are output by the function. Outputs: pandas DataFrames containing the species concentrations, loss rates, production rates, environmental outputs, photolysis rates output.

- `atchem2_path` (str): The path to an AtChem2 directory used for the model run. This path should be to the root AtChem2 directory.
- `mech_path` (str): The path to the mechanism used to build the model.
- `day` (int): Day of the month of the model start time (used for photolysis calculations).
- `month` (int): Month of the year of the model start time (used for photolysis calculations).
- `year` (int): Year of the model start time (used for photolysis calculations).
- `t_start` (int): The start time of the model in seconds. This should be in UTC time in order to properly calculate photolysis rates.
- `t_end` (int): The end time of the model in seconds. This (along with `step_size`) is used to calculate the number of model steps to specify in `model.parameters`.
- `lat` (float): latitude of the model location  (used for photolysis calculations).
- `lon` (float): longitude of the model location  (used for photolysis calculations). Note that, as is explained in the AtChem2 documentation, **longitude west is positive**. This is the opposite of the standard convention where east is positive.
- `step_size` (int): The length of each timestep in seconds. This model timestep is currently also used for the concentration and rate output timestep, meaning models will output data for every step of the model.
- `initial_concs` (pd.Series = pd.Series()): A series of initial species concentrations. The index should be a sequence of species names present in the mechanism used, and values should be the corresponding starting concentrations for each species.
- `spec_constrain` (pd.DataFrame = pd.DataFrame()): A dataframe of species constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each species that should be constrained, and the values correspond to the desired species concentration at each time. NaN can be used and will be removed for each species constraint prior to editing the configuration files.
- `spec_constant` (pd.Series = pd.Series()): A series of species that should be held at constant concentration throughout the simulation. The index should be a sequence of species names present in the mechanism used, and values should be the corresponding constant concentrations for each species.
- `env_constrain` (pd.DataFrame = pd.DataFrame()):  A dataframe of environmental constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each environmental parameter that should be constrained, and the values correspond to the desired value at each time. NaN can be used and will be removed for each environmental constraint prior to editing the configuration files.
- `photo_constant` (pd.Series = pd.Series()): A series of photolysis rates that should be held constant throughout the simulation. The index should be a sequence of J-value names (e.g. J1, J2, etc.), and values should be the corresponding constant rate for each J-value.
- `photo_constrain` (pd.DataFame = pd.DataFrame()): A dataframe of photolysis constraints for use in the simulation. The index should be a time series in seconds, the columns should be names of each J-value that should be constrained (e.g. J1, J2, etc.), and the values correspond to the desired value at each time. NaN can be used and will be removed for each photolysis constraint prior to editing the configuration files.
- `env_vals` (pd.Series = pd.Series(["298.15", "1013.25", "NOTUSED","3.91e+17", "0.41", "NOTUSED", "NOTUSED", "NOTUSED", "OPEN",  "NOTUSED"],  index = ["TEMP", "PRESS",  "RH", "H2O", "DEC", "BLHEIGHT", "DILUTE", "JFAC", "ROOF", "ASA"])): A series of environmental parameters that should be used in the simulation. The index should be a sequence of environmental parameter names, and values should be the corresponding value for each parameter. If any required environmental parameters are missing from the series, then they will be filled with default values. Will also accept non-default parameter names for custom AtChem2 versions that include additional environmental parameters.
- `spec_output` (list = []): A list of species names corresponding to desired concentration output species.
- `rate_output` (list=[]): A list of species names corresponding to desired rate output species.
- `keep_rundirs` (bool = False): Determines whether to keep temporary model directories created for the purposes of running the simulation. A unique model sub-directory name will be generated (model1, model2, model3, etc...) within the root AtChem2 directory to run the model. If this argument is `False` (default), then this temporary model sub-directory will be deleted once the model has finished running. If this argument is `True`, then the sub-directory will not be deleted.
- `uselock` (bool = False): Determines whether to use a file lock when building the model to prevent errors from simultaneous builds. There is no problem using the same atchem executable for multiple simulations (provided different `model` directories are used), but trying to build two models simultaneously can cause issues. The lock can slow down runs if there are lots of runs (e.g. >100) waiting for lock release before building.
- `injection_df` (pd.DataFrame = pd.DataFrame()): A dataframe of species concentrations at specified points throughout the simulation. This was originally created to facilitate simulation of 'injections' of species into atmospheric simulation chambers, however the concetration will be adjusted regardless of the current species concentration. This may result in an instantaneous *decrease* in species concentrations (as opposed to an increase, as would occur with a species injection) if the concentration is above the desired value. This can be thought of as similar to constraining species (e.g. using `spec_constrain`), however the species concentrations are allowed to vary freely between the defined injection concentrations.
    The index should be a time series in seconds, the columns should be names of each species that should be 'injected', and the values correspond to the desired species concentration at each time. NaN can be used and will be removed for each species constraint prior to editing the configuration files.
    This functionality works by running multiple sequential simulations, a new simulation at each 'injection' time, with the initial concentrations of each simulation dictated by the output of the previous simulation (except for the 'injected' species concentration which is adjusted to match the desired concentration). This cannot currently be used alongside `nox_series`.
- `nox_series` (pd.Series = pd.Series()): A series of NO<sub>x</sub> concentrations used to constrain total NO<sub>x</sub>  while allowing NO and NO<sub>2</sub> to partition freely. The index should be a time series in seconds, and the values should be the desired NO<sub>x</sub> concentration at each time.
    This feature is experimental and also very slow. It currently works by running a series of one step simulations with NO and NO<sub>2</sub> concentrations adjusted after each step to match the desired concentration (but maintaining the ratio of NO to NO<sub>2</sub>). This means that the output from this model currently shows incorrect total NOx concentrations for the last model timestep. 
    The NO<sub>x</sub> concentrations will be linearly interpolated along all of the model timesteps. This cannot currently be used alongside `injection_df`.

## Reading Model Output
If you run AtChem2 outside of AtChem-tools then you may find that you want to read simulation output into python for processing and/or plotting. AtChem-tools provides several functions that help to produce pandas dataframes from AtChem2 output files. These functions are defined in `AtChemTools/read_output.py`, which can be imported into your python script using `from AtChemTools import read_output`, provided you have properly exported AtChemTools to PYTHONPATH. Below is a description of the two main functions associated with reading AtChem2 output files.
### AtChemTools.read_output.species_concentrations_df
Reads in a `speciesConcentration.output` file output by AtChem2 to a pandas dataframe. Outputs: a pandas DataFrame containing the species concentrations, with species names as the columns and the model time (seconds) as the index.
- `file_path` (string): Filepath to the AtChem2 `speciesConcentration.output` file to be read.
- `species` (list or string = "ALL"): species to include in the outputted dataframe. Can be a list or a string. If `species` is a list, then each element should be a string corresping to a species name. If `species` is a string then it can either be the name of one species or "ALL". If "ALL" is passed, then all species present in the  `speciesConcentration.output` file will be included in the outputted dataframe.
- `error_for_non_species` (bool = False): Determines whether or not to raise an exception if the user requests species output that are not present in the  `speciesConcentration.output` file.

### AtChemTools.read_output.rate_df
Reads in a `lossRates.output` or `productionRates.output` file output by AtChem2 to a pandas dataframe. Outputs: a pandas DataFrame containing the rate data, with a `Multiindex` of model time (seconds), species names, and reaction numbers. The columns are the species number, the rate value, and the reaction string. 
Note that the reaction number is used in the index instead of the reaction string to avoid conflicts with duplicate reactions. Species names are unique, so do not face the same potential conflict, hence the use of species names in the index.
- `file_path` (string): Filepath to the AtChem2 rate output file to be read.
- `species` (list or string = "ALL"): species to include in the outputted dataframe. Can be a list or a string. If `species` is a list, then each element should be a string corresping to a species name. If `species` is a string then it can either be the name of one species or "ALL". If "ALL" is passed, then all species present in the rate output file will be included in the outputted dataframe.
- `drop_0` (bool = True): Controls output of rates which are 0 throughout the whole dataset. If `True` then any reactions with a rate equal to 0 for the entire time series will be excluded from the output dataframe.
- `drop_net_0` (bool = True): Controls the output for reactions where the net production or loss of a species is 0. If `True` the entries where a species appears as both a product and reactant for a given reaction will be excluded from the output dataframe.
- `drop_rev` (bool = False): Controls the output of reversible reactions. If `True` then the rates of reactions which have an analogous reverse reaction (i.e. identical but opposite reactants and products) will be excluded from the output. This can be useful for rate analysis of species with fast reversible production/loss reactions (e.g. the formation and loss of NO<sub>3</sub> by N<sub>2</sub>O<sub>5</sub>).
- `error_for_non_species` (bool = False): Determines whether or not to raise an exception if the user requests species output that are not present in the rate output file.
## Plotting Model Output
AtChem-tools currently has very inbuilt limited plotting functionality. There is one plotting function defined in `AtChemTools/plotting_functions.py`, which is described below. There is also a script at `Examples/ROPA_Plotting.py` which uses many of the `read_output` functions defined above to produce a stackplot of production and loss rates for given species.

### AtChemTools.plotting_functions.plot_species
Produces a multipanel figure with of species concentrations over time. Returns: matplotlib `Figure` object with each species time series plotted on a different subplot.
- `conc_df` (pd.DataFrame): A pandas dataframe containing species concentration information. Index should be model time in seconds and columns should be model species. This is the same format as is output from `AtChemTools.read_output.species_concentrations_df`.
- `species` (list): A list of species names to produce plots for. All species should be present in the `conc_df`, with the exception of `NOx` which can be specified to produce a plot with the sum of `NO` and `NO2`(which must be present in `conc_df`). 
- `nrows` (int or NoneType = 1): The number of rows of axes in the figure. The default is 1, meaning all axes will be places side-by-side in one row. If `None` then the number of rows will be determined based on `ncols`. `nrows` and `ncols` cannot both be set to `None`, and values passed to `nrows` and `ncols` must be able to accomodate the number of plots requested by `species`.
- `ncols` (int or NoneType = None): The number of columns of axes in the figure. The default is `None`, meaning the number of columns will be determined by `nrows`. `nrows` and `ncols` cannot both be set to `None`, and values passed to `nrows` and `ncols` must be able to accomodate the number of plots requested by `species`.
- `units` (list or NoneType = None): The units for each axis, in the same order as `species`. Input data is assumed to be in units of molecules cm<sup>-3</sup>. Accepted conversion units are `"molecules/cm3"` (no conversion applied), `"ppb"`, or `"ppt"`. Conversion to ppb and ppt is made using `cconv`. If `None` then no unit conversions are applied.
- `cconv` (float = 2.45E19): Concentration of air used to convert from concentration to mixing ratio units, if `units` is passed. Currently only supports a single conversion factor over the course of the simulation.
- `title` (string = ""): Title to apply to the figure.
- `ax_size` (float or int = 5): Size of each subplot (in inches). Passed through to the `figsize` argument of `plt.Figure()`
- `convert_xaxis` (bool = True): If `True`, converts the x-axis into datetime objects for better formatting in the final figure. 
- `xaxis_units` (string = "UTC"): Used only for labelling the x-axis of each subplot.
- `**kwargs` :  Keyword arguments passed to `matplotlib.pyplot.plot()`.

### Examples/ROPA_Plotting.py
This script can be called to produce a stackplot of production and loss reactions for given species. To use, run the script in python with command-line arguments defined below. It should be noted that the use of quotation marks when passing command-line arguments can be helpful to avoid the shell from splitting your arguments in unexpected ways (e.g. pass `"title_page_text = Test plotting"` instead of `title_page_text = Test plotting`, which would be split up by the spaces.) Produces a pdf called ''temp_rates_plot.pdf" containing the plots.

The first 5 command-line arguments must be passed in this order:
 - Model Output Path (string, path to a model output directory containing a lossRates.output file and productionRates.output file)
 - Species of Interest (Comma Separated List e.g. NO2,O3,NO3)
 - Number of Reactions to List (int, e.g. 10 for Top 10 Reactions, with the rest being lumped into "Other")
 - Start Time (In Model Time or "START")
 - End Time (In Model Time or "END")
 
Additional key word arguments (e.g. drop_rev=True) are:
 - title_page_text (string, Text to add to the title page of the pdf output.)
 - drop_rev (bool, Ignore reversible reactions? Default: False)
 - drop_0 (bool, Ignore reactions where the rate is 0 throughout the model. Default: True)
 - drop_net_0 (bool, Ignore reactions where the species of interest is both a reactant and product. Default: True)
                     



