# Prior to Production

This guide is intended to walk users through steps necessary to take before running a production simulation.

## Configuring the Model Run – Run Script

Start with an example of a run script for a low-resolution coupled simulation:  

We'll use the [template script](https://github.com/E3SM-Project/E3SM/blob/master/run_e3sm.template.sh) in the E3SM repository, which uses Perlmutter.

Create a new run script or copy an existing one (such as the template above). The path to it should be `<run_scripts_dir>/run.<case_name>.sh`

Below, we'll walk through each section of the header to the template script linked above.

`# Machine and project`

- `readonly MACHINE=`: the name of the machine you’re running on. Common machines include `chrysalis`, `pm-cpu`, and `compy`.
- `readonly PROJECT=`: SLURM project accounting (typically `e3sm`).

!!! warning
    This tells the SLURM scheduler which project to charge for the computing time. Make sure to choose the correct project and find out what that should be if you don't know.

`# Simulation`

- `readonly COMPSET=`: compset (configuration)
- `readonly RESOLUTION=`: resolution
- `readonly CASE_NAME=`: case name
- `# readonly CASE_GROUP=`: This will let you mark multiple cases as part of the same group for later processing (e.g., with PACE).

!!! warning
    If this is part of a simulation campaign, ask your group lead about using a `CASE_GROUP` label. Otherwise, please use a unique name to distinguish from existing `CASE_GROUP` label names, (e.g., “v2.LR“).

`# Code and compilation`

- `readonly CHECKOUT=`: Date the code was checked out on, in the form `{year}{month}{day}` (yyyymmdd). The source code will be checked out in `<code_source_dir>/{year}{month}{day}`.
- `readonly BRANCH=`: branch the code was checked out from. Valid options include “master”, a branch name, or a git hash. For provenance purposes, it is best to specify the git hash.
- `readonly DEBUG_COMPILE=`: option to compile with DEBUG flag (leave set to false)

!!! warning
    A case is tied to one code base and one executable. That is, if you change `CHECKOUT` or `BRANCH`, then you should also change `CASE_NAME`.

`# Run options`

- `readonly MODEL_START_TYPE=`: specify how the model should start – use initial conditions,  continue from existing restart files, branch, or hybrid (respectively the options are: initial, continue, branch, hybrid).
- `readonly START_DATE=`: model start date (yyyy-mm-dd). Typically year 1 for simulations with perpetual (time invariant) forcing or a real year for simulations with transient forcings.

`# Set paths`

- `readonly CODE_ROOT=`: where the E3SM code will be checked out. Likely, `<code_source_dir>/${CHECKOUT}`.
- `readonly CASE_ROOT=`: where the results will go. Likely, `<simulations_dir>/${CASE_NAME}`.

`# Sub-directories`

- `readonly CASE_BUILD_DIR=${CASE_ROOT}/build`: all the compilation files, including the executable.
- `readonly CASE_ARCHIVE_DIR=${CASE_ROOT}/archive`: where short-term archived files will reside.

`# Define type of run`

- `readonly run='XS_2x5_ndays'`: type of simulation to run – i.e, a short test for verification or a long production run. (See next section for details).

`# Coupler history`

- `readonly HIST_OPTION=`
- `readonly HIST_N=`

For example, if we set `HIST_OPTION="nyears"` and `HIST_N="5"` then the coupler history will be kept in 5 year increments.

`# Leave empty (unless you understand what it does)`

- `readonly OLD_EXECUTABLE=""`: this is a somewhat risky option that allows you to re-use a pre-existing executable. This is not recommended because it breaks provenance.

`# --- Toggle flags for what to do ----`

This section controls what operations the script should perform. The run_e3sm script can be invoked multiple times with the user having the option to bypass certain steps by toggling true / false.

- `do_fetch_code=`: fetch the source code from Github.
- `do_create_newcase=`: create new case.
- `do_case_setup=`: case setup.
- `do_case_build=`: compile.
- `do_case_submit=`: submit simulation.

The first time the script is called, all the flags should be set to true. Subsequently, the user may decide to bypass code checkout (`do_fetch_code=false`) or compilation (`do_case_build=false`). A user may also prefer to manually submit the job by setting `do_case_submit=false` and then invoking `./case.submit` themself.

## Testing the model

Before starting a long production run, it is *highly recommended* to perform a few short tests to verify:

1. The model starts without errors.
2. The model produces BFB (bit-for-bit) results after a restart.
3. The model produces BFB results when changing PE layout.

(1) can spare you from a considerable amount of frustration. Imagine submitting a large job on a Friday afternoon, only to discover Monday morning that the job started to run on Friday evening and died within seconds because of a typo in a namelist variable or input file.

Many code bugs can be caught with (2) and (3). While the E3SM nightly tests should catch such non-BFB errors, it is possible that you’ll be running a slightly different configuration (e.g., a different physics option) for which those tests have not been performed.

### Running Short Tests -- an example

The type of run to perform is controlled by the script variable `run`. You should typically perform at least two short tests (two different layouts, with and without restart). Let’s start with a short test using the 'S' (small) PE layout and running for 2x5 days: `readonly run='S_2x5_ndays'`. If you have not fetched and compiled the code, set all the toggle flags to true:

```shell
do_fetch_code=true
do_create_newcase=true
do_case_setup=true
do_case_build=true
do_case_submit=true
```

At this point, execute the run_e3sm script:

```shell
cd <run_scripts_dir>
./run.<case_name>.sh
```

Fetching the code and compiling it will take some time (30 to 45 minutes). Once the script finishes, the test job will have been submitted to the batch queue.

You can immediately edit the script to prepare for the second short test. In this case, we will be running for 10 days (without restart) using the 'M' (medium PE layout): `readonly run='M_1x10_ndays'`. Since the code has already been fetched and compiled, change the toggle flags:

```shell
do_fetch_code=false
do_create_newcase=true
do_case_setup=true
do_case_build=false
do_case_submit=true
```

and execute the script:

```shell
cd <run_scripts_dir>
./run.<case_name>.sh
```

Since we are bypassing the code fetch and compilation (by re-using the previous executable), the script should only take a few seconds to run and submit the second test.

!!! tip
    The short tests use separate output directories, so it is safe to submit and run multiple tests at once. If you’d like, you could submit additional test, for example 10 days with the medium 80 nodes ('M80') layout (`M80_1x10_ndays`).

#### Verrifying Results are BFB

Once the short tests are complete, we can confirm the results were bit-for-bit (BFB) the same. All the test output is located under the `tests` directory. To verify that the results are indeed BFB, we extract the global integral from the atmosphere log files (lines starting with ‘nstep, te’) and make sure that they are identical for all tests.

```shell
cd <simulations_dir>/<case_name>/tests
for test in *
do
  zgrep -h '^ nstep, te ' ${test}/run/atm.log.*.gz | uniq > atm_${test}.txt
done
md5sum *.txt
<hash>  atm_M_1x10_ndays.txt
<matching hash>  atm_M80_1x10_ndays.txt
<matching hash>  atm_S_2x5_ndays.txt
```

If the BFB check fails, you should stop here and understand why. If they succeed, you can now start the production simulation.
