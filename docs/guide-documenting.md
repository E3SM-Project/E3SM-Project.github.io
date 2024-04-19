# Guide -- Documenting

You should create a Confluence page for your model run in the relevant Confluence space. Use the [Simulation Run Template](https://acme-climate.atlassian.net/wiki/spaces/EWCG/pages/2297299190) as a template. See below for how to fill out this template.

<!-- TODO: where should the Confluence pages be made for v3 (and for each group)? -->

## Code

`code_root_dir` and `tag_name` are defined in `<run_scripts_dir>/run.<case_name>.sh` as `CODE_ROOT` and `BRANCH` respectively.

```shell
cd <code_root_dir>/<tag_name>
git log
```

The commit hash at the top is the most recent commit. Add `“<branch name>, <commit hash>”` to this section of your page.

## Configuration

`Compset` and `Res` are specified on in the PACE “Experiment Details” section. See “Performance Information” above for how to access PACE. Choose the latest job and list these settings on your page. Custom parameters should also be listed. Find these by running:

```shell
cd <run_scripts_dir>
grep -n "EOF >> user_nl" run.<case_name>.sh # Find the line numbers to look at
```

Copy the code blocks after `cat <<EOF >> user_nl_eam`, `cat << EOF >> user_nl_elm`, and `cat << EOF >> user_nl_mosart` to your page.

## Scripts

Push your `<run_scripts_dir>/run.<case_name>.sh` to the relevant GitHub repo/directory -- likely [E3SM Data Docs](https://github.com/E3SM-Project/e3sm_data_docs/tree/main/run_scripts), under `v3/original`. Then link it on this section of your page.

## Output Files

Specify the path to your output files: `<simulations_dir>/<case_name>`.

## Jobs

Fill out a table with columns for “Job”, “Years”, “Nodes”, “SYPD”, and “Notes”.

Log file names will give you the job IDs. Logs are found in `<simulations_dir>/<case_name>/run/`. If you have done short term archiving, then they will instead be in `<simulations_dir>/<case_name>/archive/logs/`.  Use `ls` to see what logs are in the directory. The job ID will be the two-part (period-separated) number after `.log.`.

PACE’s “Experiment Details” section shows `JobID` as well. In the table, link each job ID to its corresponding PACE web page. Note that failed jobs will not have a web page on PACE, but you should still list them in the table.

Use `zgrep "DATE=" <log> | head -n 1` to find the start date. Use `zgrep "DATE=" <log> | tail -n 1` to find the end date. If you would like, you can write a bash function to make this easier:

```shell
get_dates()
{
    for f in atm.log.*.gz; do
        echo $f
        zgrep "DATE=" $f | head -n 1
        zgrep "DATE=" $f | tail -n 1
 echo ""
    done
}
```

(If `zgrep` is unavailable, use `less <log>` to look at a gzipped log file. Scroll down a decent amount to `DATE=` to find the start date. Use `SHIFT+g` to go to the end of the file. Scroll up to `DATE=` to find the end date.)

In the “Years” column specify `<start> - <end>`, with each in `year-month-day` format.

To find the number of nodes, first look at the Processor # / Simulation Time chart on PACE. The x-axis lists the highest MPI rank used, with base-0 numbering of ranks. (PE layouts often don’t fit exactly `N` nodes but instead fill `N-1` nodes and have some number of ranks left over on the final node, leaving some cores on that node unused). Then, find `MPI tasks/node` in the “Experiment Details” section. The number of nodes can then be calculated as `ceil((highest MPI rank + 1)/(MPI tasks/node))`.

The SYPD (simulated years per day) is listed in PACE’s “Experiment Details” section as `Model Throughput`.

In the “Notes” section of the table, mention if a job failed or if you changed anything before re-running a job.

## Global Time Series

!!! note
    The plots will be available online at the URL corresponding to `<www>/global_time_series/` (where `www` is specified in the `zppy` cfg). See the [E3SM Diags quick guide](https://e3sm-project.github.io/e3sm_diags/_build/html/master/quickguides/quick-guide-general.html) to find the URLs for the web portals on each E3SM machine (listed as `<web_address>`).

You can download the images and then upload them to your Confluence page.

## E3SM Diags

!!! note
    The plots will be available online at the URL corresponding to `<www>/e3sm_diags/` (where `www` is specified in the `zppy` cfg). See the [E3SM Diags quick guide](https://e3sm-project.github.io/e3sm_diags/_build/html/master/quickguides/quick-guide-general.html) to find the URLs for the web portals on each E3SM machine (listed as `<web_address>`).

Replace the baseline diagnostics in the template's table with relevant ones (e.g., diags on v3 `piControl` and another relevant `v3` run). Add your own diagnostics links in the last columns, labeling them as `<start_year>-<end_year>`.

## MPAS Analysis

!!! note
    The plots will be available online at the URL corresponding to `<www>/mpas_analysis/` (where `www` is specified in the `zppy` cfg). See the [E3SM Diags quick guide](https://e3sm-project.github.io/e3sm_diags/_build/html/master/quickguides/quick-guide-general.html) to find the URLs for the web portals on each E3SM machine (listed as `<web_address>`).

Make a bulleted list of links, e.g., for `<url_path>/ts_0001-0050_climo_0021-0050/`, create a bullet `"1-50 (time series), 21-50 (climatology)"`.

## ILAMB

!!! note
    The plots will be available online at the URL corresponding to `<www>/ilamb/` (where `www` is specified in the `zppy` cfg). See the [E3SM Diags quick guide](https://e3sm-project.github.io/e3sm_diags/_build/html/master/quickguides/quick-guide-general.html) to find the URLs for the web portals on each E3SM machine (listed as `<web_address>`).
