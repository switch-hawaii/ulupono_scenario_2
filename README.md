# Introduction

This repository contains data used to construct "Ulupono Scenario 2" used in
Hawaii PUC docket 2018-0088 on Performance Based Regulation. This scenario was
created by first choosing an optimal long-term design for the power system in
5-year steps (using data in the `inputs` directory), then evaluating the
performance of the model in one-year steps (using data in the `inputs_annual`
directory). Both of these optimization problems were solved using a pre-release
version of Switch 2.0.6. Results from these two steps are in the `outputs` and
`outputs_annual` directories. These became "Ulupono Scenario 2" in the RIST
workbook used in this docket.

The sections below describe how to install the data on your computer, install
the version of Switch used to solve this model (only needed if you want to modify and re-solve the model), solve the model, and inspect the results.

Please contact Matthias Fripp <mfripp@hawaii.edu> if you have any questions
about using the data or model or interpreting the results.

# Installation steps

Note: you can complete only the first step and skip the skip the remaining steps if you only want to inspect inputs or outputs for previously solved models. The later steps tell you how to install Switch and solve the model yourself.

## Install Scenario 2 data

Use a web browser to download the data from
https://github.com/switch-hawaii/ulupono_scenario_2/archive/master.zip, then
unzip it into a directory called ulupono_scenario_2. Place this wherever you
find convenient.

## Install Python, Switch and a solver

The steps below are only needed if you want to modify and re-solve the model. If
you would just like to inspect the source code for Switch, you can find the
version used for this study at
https://github.com/switch-model/switch/tree/95c9d0f43b23a2972c7efe18a9c150c2e24bc302
.

[Miniconda](https://docs.conda.io/en/latest/miniconda.html) is a good, cross-platform installer. Alternatively, you can use the [standard Python installer](https://www.python.org/downloads/).

You should install Python 3.7 or later. Earlier versions may work but have not been tested. Either of these should include the `pip` package manager for Python.

Next, launch a Terminal window or Anaconda prompt and then install the
version of Switch used for this study, along with its dependencies, by typing
this command (all on one line):
`pip install https://github.com/switch-model/switch/archive/95c9d0f43b23a2972c7efe18a9c150c2e24bc302.zip`

In addition to these, you will need a commercial-grade solver to actually solve
the optimization model. CPLEX and Gurobi work well. Open-source solvers such as
glpk or cbc are not fast enough for this model.

Be sure to update the lines for `--solver` and `--solver-options-string` in ulupono_scenario_2/options.txt to match the solver you are using.

# Solve the model

This step is only needed if you have modified the inputs. Otherwise, you can
inspect the already-solved outputs as described in the next section.

You can solve the model as follows:

- launch a Terminal window or Anaconda Command prompt
- use the `cd` command to navigate to the directory the `ulupono_scenario_2`
  directory
- run this command: `switch solve`
  - this will solve the main optimization stage and write the results to the
    `outputs` directory
  - you can run `switch solve --help` to see more options.
- run this command: `python interpolate_construction_plan.py`
  - this will move installation of batteries and utility-scale solar to earlier
    years to smooth out the installation plan (the new plan is saved in `inputs_annual/*_modified.csv`)
- run this command: `cat outputs/BuildPumpedHydroMW.csv` (Mac/Linux) or `type outputs/BuildPumpedHydroMW.csv`
  - make note of the year when pumped hydro is built and the amount built
- run this command: `switch solve --inputs-dir inputs_annual --outputs-dir outputs_annual --ph-mw 150 --ph-year 2030 --input-alias gen_build_predetermined.csv=gen_build_predetermined_adjusted.csv  generation_projects_info.csv=generation_projects_info_adjusted.csv --exclude-module switch_model.hawaii.heco_outlook_2020_06`
  - be sure to update the `--ph-mw` and `--ph-year` to show the correct amount
    and date; use `--ph-mw 0 --ph-year 2020` if no pumped hydro is built
  - this will save the annual results in `outputs_annual`

Note that re-solving the model may produce different results from the ones shown
in the repository. This is because the model is generally solved only to within
0.5% of perfect optimality, and a variety of solutions are possible within this
limit. Different solutions may be found by different solvers or different
versions of the same solver, or even the same solver running with different
flags or on different machines. However, all the solutions should be within 0.5%
of each other in total cost.

# Inspect results

All results from the main optimization stage (with investment decisions in 2020, 2023, 2025, 2030, 2035, 2040, 2045 and 2050) are in `outputs`. All results from the annual evaluation are in `outputs_annual`. The files with mixed capital and lowercase names (e.g., `BuildGen.csv`) are individual decisions made by the model. The files with lowercase names and underscores (e.g., `capacity_by_technology.csv`) are summary files. `total_cost.txt` shows the NPV of total costs in 2020-2054.

Here are some useful tables:

- `capacity_by_technology.csv`: summary of capacity in place each year
- `production_by_technology.csv`: summary of energy sources used each year
- `annual_details_by_tech.csv`: data used for RIST workbook in Hawaii PUC docket
  2018-0088
- `gen_dispatch.csv`: details on dispatch of individual standard generators
- `PumpedHydroProjGenerateMW.csv`: details on dispatch of pumped hydro (if any)
