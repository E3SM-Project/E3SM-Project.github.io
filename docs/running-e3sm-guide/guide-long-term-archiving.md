# Long-Term Archiving

Simulations that are deemed sufficiently valuable should be archived using `zstash` for long-term preservation. You can ask questions about `zstash` on the [zstash discussion board](https://github.com/E3SM-Project/zstash/discussions/categories/questions).

## 1. Clean up directory

Log into the machine that you ran the simulation on. Remove all `eam.i` files except the latest one. Dates are of the form `<YYYY-MM-DD>`.

```shell
cd <simulations_dir>/<case_name>/run
ls | wc -l # See how many items are in this directory
mv <case_name>.eam.i.<Latest YYYY-MM-DD>-00000.nc tmp.nc
rm <case_name>.eam.i.*.nc
mv tmp.nc <case_name>.eam.i.<Latest YYYY-MM-DD>-00000.nc
ls | wc -l # See how many items are in this directory
```

There may still be more files than is necessary to archive. You can probably remove `*.err`, `*.lock`, `*debug_block*`, `*ocean_block_stats*` files.

## 2. `zstash create` & Transfer to NERSC HPSS

On the machine that you ran the simulation on:

If you don’t have one already, create a directory for utilities, e.g., `utils/`. Then, open a file in that directory called `batch_zstash_create.bash` and paste the following in it, making relevant edits:

```shell
#!/bin/bash
# Run on <machine name>
# Load E3SM Unified
<Command to load the E3SM Unified environment>
# List of experiments to archive with zstash
EXPS=(\
<case_name> \
)
# Loop over simulations
for EXP in "${EXPS[@]}"
do
    echo === Archiving ${EXP} ===
    cd <simulations_dir>/${EXP}
    mkdir -p zstash
    stamp=`date +%Y%m%d`
    time zstash create -v --hpss=globus://nersc/home/<first letter>/<username>/E3SMv2/${EXP} --maxsize 128 . 2>&1 | tee zstash/zstash_create_${stamp}.log
done
```

!!! tip
    Compy, Anvil and Chrysalis do not have local HPSS. We rely on NERSC HPSS for long-term archiving. If you are archiving a simulation run on Compy or LCRC (Chrysalis/Anvil), we need to include the Globus piece above. If you are archiving a simulation run on NERSC (Perlmutter), you can simplify that piece to `--hpss=E3SMv2/${EXP}`

Load the E3SM Unified environment.
!!! tip
    The E3SM Unified environment activation commands can be found on [zppy's Getting started page](https://docs.e3sm.org/zppy/_build/html/main/getting_started.html). Alternatively, they can be found using [Mache](https://github.com/E3SM-Project/mache/tree/main/mache/machines): click the relevant machine and find the `base_path` listed under `[e3sm_unified]` -- the activation command will be `source <base_path>/load_latest_e3sm_unified_<machine_name>.sh`.

Then, do the following:

```shell
$ screen # Enter screen
$ screen -ls # Output should say "Attached"
$ ./batch_zstash_create.bash 2>&1 | tee batch_zstash_create.log
# Control A D to exit screen
# DO NOT CONTROL X / CONTROL C (as for emacs). This will terminate the task running in screen!!!
$ screen -ls # Output should say "Detached"
$ hostname
# If you log in on another login node, 
# then you will need to ssh to this one to get back to the screen session.
$ tail -f batch_zstash_create.log # Check log without going into screen
# Wait for this to finish
$ screen -r # Return to screen
# Check that output ends with `real`, `user`, `sys` time information
$ exit # Terminate screen
$ screen -ls # The screen should no longer be listed
$ ls <simulations_dir>/<case_name>/zstash
# `index.db`, and a `zstash_create` log should be present
# No tar files should be listed
# If you'd like to know how much space the archive or entire simulation use, run:
$ du -sh <simulations_dir>/<case_name>/zstash
$ du -sh <simulations_dir>/<case_name>
```

Then, on NERSC/Perlmutter:

```shell
$ hsi
$ ls /home/<first letter>/<username>/E3SMv2/<case_name>
# Tar files and `index.db` should be listed.
# Note `| wc -l` doesn't work on hsi
$ exit
```

## 3. `zstash check`

On a NERSC machine (Perlmutter):

```shell
cd /global/homes/<first letter>/<username>
emacs batch_zstash_check.bash
```

Paste the following in that file, making relevant edits:

```shell
#!/bin/bash
# Run on NERSC dtn
# Load environment that includes zstash
<Command to load the E3SM Unified environment>
# List of experiments to archive with zstash
EXPS=(\
<case_name> \
)
# Loop over simulations
for EXP in "${EXPS[@]}"
do
    echo === Checking ${EXP} ===
    cd /global/cfs/cdirs/e3sm/<username>/E3SMv2
    mkdir -p ${EXP}/zstash
    cd ${EXP}
    stamp=`date +%Y%m%d`
    time zstash check -v --hpss=/home/<first letter>/<username>/E3SMv2/${EXP} --workers 2 2>&1 | tee zstash/zstash_check_${stamp}.log
done
```

!!! tip
    If you want to check a long simulation, you can use the `--tars` option to split the checking into more manageable pieces:
  
    <!-- markdownlint-disable MD046 -->
    ```shell
    # Starting at 00005a until the end
    zstash check --tars=00005a-
    # Starting from the beginning to 00005a (included)
    zstash check --tars=-00005a
    # Specific range
    zstash check --tars=00005a-00005c
    # Selected tar files
    zstash check --tars=00003e,00004e,000059
    # Mix and match
    zstash check --tars=000030-00003e,00004e,00005a-
    ```
    <!-- markdownlint-restore -->

Then, do the following:

```shell
$ ssh dtn01.nersc.gov
$ screen
$ screen -ls # Output should say "Attached"
$ cd /global/homes/<first letter>/<username>
$ ./batch_zstash_check.bash
# Control A D to exit screen
# DO NOT CONTROL X / CONTROL C (as for emacs). This will terminate the task running in screen!!!
$ screen -ls # Output should say "Detached"
$ hostname
# If you log in on another login node, 
# then you will need to ssh to this one to get back to the screen session.
# Wait for the script to finish
# (On Chrysalis, for 165 years of data, this takes ~5 hours)
$ screen -r # Return to screen
# Check that output ends with `INFO: No failures detected when checking the files.`
# as well as listing real`, `user`, `sys` time information
$ exit # Terminate screen
$ screen -ls # The screen should no longer be listed
$ exit # exit data transfer node
$ cd /global/cfs/cdirs/e3sm/<username>/E3SMv2/<case_name>/zstash
$ tail zstash_check_<stamp>.log
# Output should match the output from the screen (without the time information)
```

## 4. Document

On a NERSC machine (Perlmutter):

```shell
$ hsi
$ ls /home/<first letter>/<username>/E3SMv2
# Check that the simulation case is now listed in this directory
$ ls /home/<first letter>/<username>/E3SMv2/<case_name>
# Should match the number of files in the other machine's `<simulations_dir>/<case_name>/zstash`
$ exit
$ cd /global/cfs/cdirs/e3sm/<username>/E3SMv2/<case_name>/zstash
$ ls
# `index.db` and `zstash_check` log should be the only items listed
# https://www2.cisl.ucar.edu/resources/storage-and-file-systems/hpss/managing-files-hsi
# cput will not transfer a file if it exists.
$ hsi "cd /home/<first letter>/<username>/E3SMv2/<case_name>; cput -R <zstash_check log>"
$ hsi
$ ls /home/<first letter>/<username>/E3SMv2/<case_name>
# tar files, `index.db`, `zstash_create` log, and `zstash_check` log should be present
$ exit
```

If you are E3SM staff, update the (internal) simulation Confluence page with information regarding this simulation (For v3 work, that page is [V3 Simulation Planning](https://acme-climate.atlassian.net/wiki/spaces/ED/pages/4282679297/V3+Simulation+Planning)). In the `zstash archive` column, specify:

- `/home/<first letter>/<username>/E3SMv2/<case_name>`
- `zstash_create_<stamp>.log`
- `zstash_check_<stamp>.log`

<!-- TODO: Is there a v3 log page to link? -->

## 5. Delete files

On a NERSC machine (Perlmutter):

```shell
$ hsi
$ ls /home/<first letter>/<username>/E3SMv2/<case_name>
# tar files, `index.db`, `zstash_create` log, and `zstash_check` log should be present
# So, we can safely delete these items on cfs
$ exit
$ ls /global/cfs/cdirs/e3sm/<username>/E3SMv2/<case_name>
# Should match output from the `ls` on `hsi` above
$ cd /global/cfs/cdirs/e3sm/<username>
$ ls E3SMv2
# Only the <case_name> you just transferred to HPSS should be listed.
# We only want to remove that one.
$ rm -rf E3SMv2
```

On the machine that you ran the simulation on:

```shell
$ cd <simulations_dir>/<case_name>
$ ls zstash
# tar files, index.db, `zstash_create` log should be present
$ rm -rf zstash # Remove the zstash directory, keeping original files
$ cd <simulations_dir>
```

## More info

Refer to [zstash's best practices for E3SM](https://docs.e3sm.org/zstash/_build/html/master/best_practices.html) for details.
