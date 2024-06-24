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

## Processing configs 
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
To start the processing run:
``` 
julia -t 1 --project=/ptmp/lschl/l200/current/jlenv --heap-size-hint=10G main.jl --config ./config/processing_config.json -p 3 -r 0 1 --only_runs  
```
* `-t 1` number of threads (?)
* `--project=/ptmp/lschl/l200/current/jlenv` define project environment
* `--heap-size-hint=10G` setting for garbage collector
* `main.jl` **This is the function that is executed!**
* `--config ./config/processing_config.json` **important:** tells main.jl, which configuration to use
* `-p 3 -r 0 1 --only_runs` optional: come overwrites for the config file


This will open new workers. No actural CPU cores are assigned to these workers yet.  The assignment happens via `startjlworkers.sh`, that is automatically created in the user's home directory. 










