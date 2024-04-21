# Appendix

This appendix details some specifities and helpful tips.

## Useful Aliases

Setting some aliases may be useful in running the model. You can edit your bash settings file to add aliases. Run `source` on that file to start using your new aliases. Examples:

- Chrysalis – `source ~/.bashrc`
- Compy – `source ~/.bash_profile`
- Perlmutter – `source ~/.bash_profile.ext`

Note that the specific file name may differ amongst machines. For example, it might be named `~/.bashrc.ext`.

### Batch Jobs

To check on all batch jobs:

`alias sqa='squeue -o "%8u %.7a %.4D %.9P %7i %.2t %.10r %.10M %.10l %.8Q %j" --sort=P,-t,-p'`

To check on your batch jobs:

`alias sq='sqa -u $USER'`

The output of `sq` uses several abbreviations: ST = Status, R = running, PD = pending, CG = completing.

### Directories

You will likely be working in several directories. Here are some example values:

`<run_scripts_dir>`: `${HOME}/E3SM/scripts`

`<code_source_dir>`: `${HOME}/E3SM/code`

The path for `<simulations_dir>` will likely be machine-dependent. For example,:

- Anvil/Chrysalis (LCRC): `/lcrc/group/e3sm/<username>/E3SMv2`
- Compy (PNNL): `/compyfs/<username>/E3SMv2`
- Perlmutter (NERSC): `/global/cfs/cdirs/e3sm/<username>/E3SMv2`

So, it may be useful to set the following aliases:

```shell
# Model running

alias run_scripts="cd <run_scripts_dir>"
alias code_source="cd <code_source_dir>"
alias simulations="cd <simulations_dir>"
```
