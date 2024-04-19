# Guide -- Post-Processing

This guide is intended to walk users step-by-step through post-processing a simulation.

## Short Term Archiving

First, short-term archiving is quite useful for post-processing.

By default, E3SM will store all output files under the `<simulations_dir>/<case_name>/run/` directory. For long simulations, there could 10,000s to 100,000s of output files. Having so many files in a single directory can be very impractical, slowing down simple operations like `ls` to a crawl. CIME includes a short-term archiving utility that will neatly organize output files into a separate `<simulations_dir>/<case_name>/archive/` directory. Short term archiving can be accomplished with the following steps.

!!! tip
    This can be done while the model is still running.

Use `--force-move` to move instead of copying, which can take a long time. Set `--last-date` to the latest date in the simulation you want to archive. You do not have to specify a beginning date.

```shell
cd <simulations_dir>/<case_name>/case_scripts
./case.st_archive --last-date <yyyy-mm-dd> --force-move --no-incomplete-logs
ls <e3sm_simulations_dir>/<case_name>/archive
```

Each component of the model has a directory under `archive/`. There are also two additional directories under `archive/`: `logs` holds the gzipped log files and `rest` holds the restart files.

| Component | Directory | File naming pattern |
| --- | --- | --- |
| Atmosphere (Earth Atmospheric Model) | `archive/atm/hist` | `*.eam.h*` |
| Coupler | `archive/cpl/hist` | `*.cpl.h*` |
| Sea Ice (MPAS-Sea-Ice) | `archive/ice/hist` | `*.mpassi.hist.*` |
| Land (Earth Land Model) | `archive/lnd/hist` | `*.elm.h*` |
| Ocean (MPAS-Ocean) | `archive/ocn/hist` | `*.mpaso.hist.*` |
| River Runoff (MOSART) | `archive/rof/hist` | `*.mosart.h*` |

## Post-Processing with zppy

To post-process a model run, do the following steps.

!!! warning
    To post-process up to year *n*, then you must have short-term archived up to year *n*.

You can ask questions about `zppy` on the [zppy discussion board](https://github.com/E3SM-Project/zppy/discussions/categories/questions).

### Install zppy

Load the E3SM Unified environment.

!!! tip
    The E3SM Unified environment activation commands can be found on [zppy's Getting started page](https://e3sm-project.github.io/zppy/_build/html/main/getting_started.html). Alternatively, they can be found using [Mache](https://github.com/E3SM-Project/mache/tree/main/mache/machines): click the relevant machine and find the `base_path` listed under `[e3sm_unified]` -- the activation command will be `source <base_path>/load_latest_e3sm_unified_<machine_name>.sh`.

If you need a feature in `zppy` that has not yet been included in the E3SM Unified environment, you can construct a [development environment](https://e3sm-project.github.io/zppy/_build/html/main/getting_started.html#b-development-environment).

### Configuration File

In `<run_scripts_dir>`, create a new post-processing configuration file, or copy an existing one, and call it `post.<case_name>.cfg`.

!!! tip
    Good example configuration files can be found in the `zppy` [integration test directory](https://github.com/E3SM-Project/zppy/tree/main/tests/integration/generated) -- `test_complete_run_<machine_name>.cfg

Edit the file and customize as needed. The file is structured with `[section]` and `[[sub-sections]]`. There is a `[default]` section, followed by additional sections for each available zppy task (`climo`, `ts`, `e3sm_diags`, `mpas_analysis`, …). Sub-sections can be used to have multiple instances of a particular task, for example having both regridded monthly and globally averaged time series files. Refer to the `zppy` [schematics documentation](https://e3sm-project.github.io/zppy/_build/html/main/schematics.html) for more details.

The key sections of the configuration file are:

`[default]`

- `input`, `output`, `www` paths will likely need to be edited.

!!! note
    The output of your simulation (`<simulations_dir>/<case_name>`) is the *input* to `zppy`. You can use the same directory for `zppy` output as well, since `zppy` will generate output under `<output>/post`

`[climo]`

- `mapping_file` path may need to be edited.
- Typically you want to generate climatology files every 20,50 years: `years = begin_year:end_yr:averaging_period` – e.g., `years = "1:80:20", "1:50:50",`.

`[ts]`

- `mapping_file` path may need to be edited.
- Typically you want to generate time series files every 10 years – e.g., `years = "1:80:10"`.

`[e3sm_diags]`

- `reference_data_path` may need to be edited.
- `short_name` is a shortened version of the case_name
- `years` should match the `[climo]` section `years`

`[mpas_analysis]`
Years can be specified separately for time series, climatology, and ENSO plots. The lists must have the same lengths and each entry will be mapped to a realization of `mpas_analysis`:

```shell
climo_years ="21-50", "51-100",
enso_years = "11-50", "11-100",
ts_years = "1-50", "1-100",
```

In this particular example, MPAS Analysis will be run twice. The first realization will produce
climatology plots averaged over years 21-50, ENSO plots for years 11 to 50, and time series plots covering years 1 to 50. The second realization will cover years 51-100 for climatologies, 11-100 for ENSO, and 1-100 for time series.

`[global_time_series]`

- `climo_years` and `ts_years` should match their equivalents in the `[mpas_analysis]` section.

!!! tip
    See the `zppy` [parameters documentation](https://e3sm-project.github.io/zppy/_build/html/main/parameters.html) for more information on parameters.

### Launch zppy

Run `zppy -c post.<case_name>.cfg`. This will submit a number of jobs. Run `sq` to see what jobs are running.

`zppy` automatically handles dependencies of jobs. E.g., `e3sm_diags` jobs are dependent on `climo` and `ts` jobs, so they wait for those to finish. MPAS Analysis jobs re-use computations, so they are chained.

Most jobs run quickly, though E3SM Diags may take around an hour and MPAS Analysis may take several hours.

`zppy` creates a new directory `<simulations_dir>/<case_name>/post`. Each realization will have a shell script (typically `bash`). This is the actual file that has been submitted to the `batch` system. There will also be a log file `*.o<job ID>` as well as a `*.status` file. The status file indicates the state (WAITING, RUNNING, OK, ERROR). These files can be found in `<simulations_dir>/<case_name>/post/scripts`. Once all the jobs are complete, you can check their status.

```shell
cd <simulations_dir>/<case_name>/post/scripts
cat *.status # should be a list of "OK"
grep -v "OK" *.status # lists files without "OK"
```

If you re-run `zppy`, it will check the status of tasks and will skip a task if its status is “OK”. As your simulation progresses, you can update the post-processing years in the configuration file and re-run `zppy`. Newly added task will be submitted, while previously completed ones will be skipped.

### Tasks

If you run `ls <simulations_dir>/<case_name>/post/scripts` you’ll see files like `e3sm_diags_180x360_aave_model_vs_obs_0001-0020.status`. This is one e3sm_diags job. Parts of this file name are explained below:

| Part of File Name | Meaning |
| --- | --- |
| `e3sm_diags` | Task |
| `180x360_aave` | Grid |
| `model_vs_obs` | `model_vs_model` or `model_vs_obs` |
| `0001-0020` | First and last years |

There is also a corresponding output file. It will have the same name but end with `.o<job ID>` instead of `.status`.

### Output

The post-processing output is organized hierarchically. Examples:

- `<e3sm_simulations_dir>/<case_name>/post/atm/180x360_aave/ts/monthly/10yr` has the time series files – one variable per file, in 10 year periods as defined in `<run_scripts_dir>/post.<case_name>.cfg`.  
- `<e3sm_simulations_dir>/<case_name>/post/atm/180x360_aave/clim/20yr` similarly has climatology files for 20 year periods, as defined in `<run_scripts_dir>/post.<case_name>.cfg``.
- `<e3sm_simulations_dir>/<case_name>/post/atm/glb/ts/monthly/10yr` has globally averaged files for 10 years periods as defined in `<run_scripts_dir>/post.<case_name>.cfg`.
