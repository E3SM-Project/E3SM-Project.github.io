# Guide -- Production

This guide is intended to walk users through steps necessary to run a production simulation.

## Running the Model

To prepare for the long production simulation, edit the run_e3sm script and set `readonly run='production'`. In addition, you may need to customize some variables in the code block below to configure run options. Below is an example of how you might configure these variables:

`# Production simulation`

- `readonly PELAYOUT="L"`: 1=single processor, S=small, M=medium, L=large, X1=very large, X2=very very large. Production simulations typically use M or L. The size determines how many nodes will be used. The exact number of nodes will differ amongst machines.
- `readonly WALLTIME="34:00:00"`: maximum wall clock time requested for the batch jobs.
- `readonly STOP_OPTION="nyears"`: see next line
- `readonly STOP_N="50"`: units and length of each segment (i.e., each batch job). E.g, the current configuration stops after 50 years.
- `readonly REST_OPTION="nyears"`: see next line
- `readonly REST_N="5"`: units and frequency for writing restart files (make sure `STOP_N` is a multiple of `REST_N`, otherwise the model will stop without writing a restart fie at the end). E.g., the current configurations saves restart files after every 5 years. 10 restart files will be saved, since `STOP_N=50`.
- `readonly RESUBMIT=”9”`: number of resubmissions beyond the original segment. This simulation would run for a total of 500 years (=inital 50 + 9x50).
- `readonly DO_SHORT_TERM_ARCHIVING=false`: leave set to false if you want to manually run the short term archive.

Since the code had already been fetched and compiled for the [short tests](guide-prior-to-production.md), the toggle flags can be set to:

```shell
do_fetch_code=false
do_create_newcase=true
do_case_setup=true
do_case_build=false
do_case_submit=true
```

Finally, execute the script:

```shell
cd <run_scripts_dir>
./run.<case_name>.sh
```

The script will automatically submit the first job. New jobs will be automatically be resubmitted at the end until the total number of segments have been run.

## Looking at Results

`ls <simulations_dir>/<case_name>` explanation of directories:

- `build`: all the stuff to compile. The executable (`e3sm.exe`) is also there.
- `case_scripts`: the files for your particular simulation.
- `run`: where all the output will be. Most components (atmosphere, ocean, etc.) have their own log files. The coupler exchanges information between the components. The top level log file will be of the form `run/e3sm.log.*`.  Log prefixes correspond to components of the model:
  - `atm`: atmosphere
  - `cpl`: coupler
  - `ice`: sea ice
  - `lnd`: land
  - `ocn`: ocean
  - `rof`: river runoff

Run `tail -f run/<component>.log.<latest log file>` to keep up with a log in real time.

You can use the `sq` alias defined in [Useful Aliases](guide-appendix.md) to check on the status of the job. The `NODE` in the output indicates the number of nodes used and is dependent on the `processor_config` / `PELAYOUT` size.  

!!! note
    When running on two different machines (such as Compy and Chrysalis) and/or two different compilers, the answers will not be the same, bit-for-bit. It is not possible using floating point operations to get bit-or-bit identical results across machines/compilers.

Logs being compressed to `.gz` files is one of the last steps before the job is done and will indicate successful completion of the segment. `less <log>.gz` will let you directly look at a gzipped log.

## Re-Submitting a Job After a Crash

If a job crashes, you can rerun with:

```shell
cd <simulations_dir>/<case_name>/case_scripts
# Make any changes necessary to avoid the crash
./case.submit
```

If you need to change a XML value, the following commands in the `case_scripts` directory are useful:

```shell
> ./xmlquery <variable>                   # Get value of a variable
> ./xmlchange -id <variable> -val <value> # Set value of a variable
```

Before re-submitting:

- Check that the rpointer files all point to the last restart. On very rare occasions, there might be some inconsistency if the model crashed at the end. Run `head -n 1 rpointer.*` to see the restart date.
- `gzip` all the `*.log` files from the faulty segment so that they get moved during the next short-term archiving. To `gzip` log files from failed jobs, run `gzip *.log.<job ID>*` (where `<job ID>` has no periods/dots in it).
- Delete core or error files, if there are any. MPAS components will sometimes produce a large number of them. The following commands are useful for checking for these files:
  - `ls | grep -in core`
  - `ls | grep -in err`
- If you are re-submitting the *initial* job, you will need to run `./xmlchange -id CONTINUE_RUN -val TRUE`.

## Performance Information

Model throughput is the number of simulated years per day (SYPD). You can find this with:

```shell
cd <simulations_dir>/<case_name>/case_scripts/timing
grep "simulated_years" e3sm*
```

PACE provides detailed performance information. Go to [PACE](https://pace.ornl.gov/) and enter your username to search for your jobs. You can also simply search by providing the JobID appended to log files (`NNNNN.yymmdd-hhmmss` where `NNNNN` is the SLURM job id). Click on a job ID to see its performance details. “Experiment Details” are listed at the top of the job’s page. There is also a helpful chart detailing how many processors and how much time each component (`atm`, `ocn`, etc.) used. White areas indicate time spent idle/waiting. The area of each box is essentially the "cost = simulation time * number of processors" of the corresponding component.
