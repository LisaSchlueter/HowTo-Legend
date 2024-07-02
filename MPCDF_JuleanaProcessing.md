# LEGEND Juleana data processing - Setup at MPCDF
This is an explaination of how to set up your own/custom Juleana data processing at the MPCDF. In case you havn't used Juleana at MPCDF (or any other supercomputer), read *MPCDF_JuleanaSetup.md* first. 

## Create your configuration file `config.json` 
* Create a configuration file `config.json`, that sets all the paths for raw files, par files, plots, logs, etc.  

* These paths are used in the data processing by *LegendDataManagement* module. A possible location to store this file is `/ptmp/$USER/l200/current/config.json` (replace `$USER` with your username). 

* Once config file is created, change `LEGEND_DATA_CONFIG` (see *MPCDF_JuleanaSetup.md*: "Accessing LEGEND Data")

Example `config.json` file:
```
{
    "setups": {
        "l200": {
            "paths": {
                "metadata": "$_/legend-metadata/",

                "tier": "$_/generated/tier/",
                "tier/raw": "/ptmp/oschulz/legend/data/l200/raw-compressed",
                "tier/jlpeaks": "/ptmp/oschulz/legend/data/l200/current/generated/tier/jlpeaks",
                "tier/jlml": "/ptmp/oschulz/legend/data/l200/current/generated/tier/jlml",
                "tier/jllog": "$_/generated/jllog/",
                "tier/jlreport": "$_/generated/jlreport/",
                "tier/jlplt": "$_/generated/jlplt/",

                "par": "$_/generated/jlpar/"
            }
        }
    }
}
```
## Clone legend-metadata (metadata configs)
* Clone [legend-metadata](https://github.com/legend-exp/legend-metadata) in the same directory as the `config.json` file (see above). Also load all submodules. Use `dev` branch. 

* The directory `legend-metadata/jldataprod/config/` contains configuration files for the juleana data processing. These files containts instructions for the processors, for example file calibration lines are supposed to be fit. 

## Processing config
* The juleana data production can be found in [legend-julia-dataflow](https://github.com/legend-exp/legend-julia-dataflow). Clone to any directory in home folder. **Use `dev` branch**. 

* In the directory `legend-julia-dataflow/config/` you can find the configuration of the processing. Here you can choose, for example, which partitions/runs you want to process. An example:
```
   "processing": {
        "periods": [3,4],
        "runs" : [1,2], 
        "partitions": [1],
        "analysis_runs_only": true,
        "only_partitions": false,
        "only_runs": false,
        "submit_slurm": false,
        "remote_workers": []
    },
```

## Create processing environment. 
Create an environment that is always used for the data production:
* `mkdir /ptmp/$USER/l200/current/jlenv`
* in julia package manager: `activate /ptmp/$USER/l200/current/jlenv` to create a new environment 
* if available, copy `Project.toml` and `Manifest.toml` in jlenv directory

## Start processing
### 1. Start the processing by running something like:
``` 
julia -t 1 --project=/ptmp/lschl/l200/current/jlenv --heap-size-hint=10G main.jl --config ./config/processing_config.json -p 3 -r 0 1 --only_runs  
```
* `-t 1` number of threads (?)
* `--project=/ptmp/lschl/l200/current/jlenv` define project environment
* `--heap-size-hint=10G` setting for garbage collector
* `main.jl` **This is the function that is executed!**
* `--config ./config/processing_config.json` **important:** tells main.jl, which configuration to use
* `-p 3 -r 0 1 --only_runs` optional: come overwrites for the config file

This will open new workers. No actual CPU cores are assigned to these workers yet.  The assignment happens via `startjlworkers.sh`, that is automatically created in the user's home directory. 
### 2. Assign computing nodes to workers
To give the workers computing power, run the bash script `startjlworkers.sh` that is in your home directory. The script can be run multiple times to add additional nodes to workers. The default number of nodes is defined in the processing config. The SLURM-relevant parts looks like this: 
```
        "slurm_settings": [
            "--time=04:00:00",
            "--mem-bind=local",
            "--threads-per-core=1",
            "--cpu-bind=cores",
            "--nodes=32",
            "--ntasks-per-node=36",
            "--cpus-per-task=2",
            "--mem=256GB"
        ]
```
* `--time` 
* `--mem-bind` 
* `--threads-per-code` 
* `--cpu-bind`
* `--nodes` number of nodes to be used. should be a natural number.
* `--ntasks-per-node` number of tasks per node 
* `--cpus-per-task` number of CPU  per task
    * *Comment:* `ntasks-per-node` x `cpus-per-task` *gives the number of CPUs per node. This value should match the hardware configuration, e.g. raven has 72 CPU per node.* 
* `--mem` memory RAM per node. 
    * *Should match hardware configuration (256 GB for raven)*

The hardware configuration of the server can be found online, e.g. for [raven](https://docs.mpcdf.mpg.de/doc/computing/raven-user-guide.html): 
* 1592 (CPU) compute nodes (of course, they are not all available)
* Each node:
    * min. 256 GB memory (RAM)
    * 72 CPU cores with 2 threads each
### Useful bash commands when using SLURM:
* `squeue -u <username>` lists your jobs, job-id and their status
* `scancel <jobid>` cancel job
* `sinfo | grep idle` overview of available nodes 

## tmux session
If run from a regular bash, the processing will be terminated if the connection to the server is lost, e.g. when closing VS-Code. To avoid that, it makes sense to start the processing from a `tmux` session. Some useful commands are below. For more see [tmux cheat sheet](https://tmuxcheatsheet.com/).  
* `tmux ls` lists all tmux sessions
* `tmux new -s <session-name>` start a new session
* `tmux kill-session` kill all sessions
* `tmux kill-ses -t <session-name>` kill a particular session
* `tmux a` attach to last session
* `tmux a -t <session-name` attach to a particular session
* `Ctrl` + `b` `d` to exit (detach from) a session
* `Ctrl` + `b` `c` create new window within a session


    














