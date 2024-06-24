# Juleana setup at MPCDF
This is an explaination of how to set up the Juleana analysis at the MPCDF. 
 
## Using ssh 
MPCDF has two supercomputers; **raven** and **cobra**. 
These machines can be accessed within the MPCDF network. 
To access them from outside, you need to use a proxy jump through a gate. 

You can do this by putting the following code in your ssh config. 

``` config
Host cobra
	HostName cobra.mpcdf.mpg.de
	User <username>
	ProxyCommand ssh -W %h:%p gate1.mpcdf.mpg.de 2>/dev/null
	PubkeyAuthentication no
	ForwardAgent yes
	ServerAliveInterval 120
	GSSAPIAuthentication yes
	GSSAPIDelegateCredentials yes

Host raven
	HostName raven.mpcdf.mpg.de
	User <username>
	ProxyCommand ssh -W %h:%p gate1.mpcdf.mpg.de 2>/dev/null
	PubkeyAuthentication no
	ForwardAgent yes
	ServerAliveInterval 120
	GSSAPIAuthentication yes
	GSSAPIDelegateCredentials yes

Host gate1.mpcdf.mpg.de
	User <username>
	GSSAPIAuthentication yes
	GSSAPIDelegateCredentials yes
	ControlMaster auto
	ControlPath ~/.ssh/control:%h:%p:%r
```
More information about MPCDF can be found here: 
[cobra](https://www.mpcdf.mpg.de/services/supercomputing/cobra)

[raven](https://www.mpcdf.mpg.de/services/supercomputing/raven)

## Storage
There is limited storage in your home directory, with a limited number of files.
Your home directory should be used for code and for developing work. 

It is possible to save temporarily generated files to `/ptmp` for quickly accessing large files. 
However, files saved to `/ptmp` will be deleted after they are not accessed for six weeks. 

Long term storage of large files has not been fully resolved yet. 
For the most up to date information, please contact Oliver Schulz: oschulz@mpp.mpg.de. 

## Setup Julia 
First, create folders in `/ptmp` with the following code: 
``` bash 
mkdir -p "$HOME/.julia"
mkdir -p "/ptmp/$USER/.julia"/{artifacts,clones,compiled,conda,juliaup,logs,packages,registries,scratchspaces}
ln -s "/ptmp/$USER/.julia"/{artifacts,clones,compiled,conda,juliaup,logs,packages,registries,scratchspaces} "$HOME/.julia/"
touch "$HOME/.julia/repl_history.jl"
ln -s "$HOME/.julia/repl_history.jl" "/ptmp/$USER/.julia/logs/"
```

Next it is recommended to install Julia at MPCDF using [juliaUp](https://github.com/JuliaLang/juliaup). 
To do this, use the following code: 
``` bash 
curl -fsSL https://install.julialang.org | sh
```
Afterwards, include the following code in your `~/.bashrc` file, where `$USER` is your username: 
``` bash 
# >>> juliaup initialize >>>

# !! Contents within this block are managed by juliaup !!

case ":$PATH:" in
    *:/u/$USER/.juliaup/bin:*)
        ;;

    *)
        export PATH=/u/$USER/.juliaup/bin${PATH:+:${PATH}}
        ;;
esac

# <<< juliaup initialize <<<
```
Run `exec bash` and attempt to open julia by running the command `julia` to check that it has been successfully installed. 

## Setup LEGEND Julia Registry 
Execute the following command to set up the LEGEND Julia registry: 
``` julia
include(download("https://raw.githubusercontent.com/legend-exp/legend-julia-tutorial/main/legend_julia_setup.jl"))
```

This registry is necessary as it allows you to install the Juleana packages. 

## Julia and SLURM

SLURM is the program used to submit jobs on supercomputer systems and is necessary for efficient processing. 

Set up Julia to run correctly with SLURM by replacing the code in the file `~/.julia/config/startup.jl` with the following:
``` julia 
if get(ENV, "JULIA_REVISE", "") != "off"
    try
        using Revise
    catch e
        @warn "Error initializing Revise, try adding Revise.jl to your environment" exception=(e, catch_backtrace())
    end
end
@eval Module() begin
    try
        import Pkg;
        Pkg.UPDATED_REGISTRY_THIS_SESSION[] = true
    catch e
        @warn "Can't set Pkg.UPDATED_REGISTRY_THIS_SESSION" exception=(e, catch_backtrace())
    end
end
if haskey(ENV, "NERSC_HOST")
    @eval Module() begin
        import Sockets
        permutter_hsn_addr = Sockets.IPv4(filter(!isnothing, match.(r"inet (.*)/.*hsn0", readlines(`ip a show`)))[1].captures[1])
        Sockets.getipaddr() = permutter_hsn_addr
    end
elseif get(ENV, "CLUSTER", "") == "COBRA"
    @eval Module() begin
        import Sockets
        cobra_infiniband_addr = Sockets.IPv4(filter(!isnothing, match.(r"inet (.*)/.*ib0", readlines(`ip a show`)))[1].captures[1])
        Sockets.getipaddr() = cobra_infiniband_addr
        if haskey(ENV, "JOB_TMPDIR")
            ENV["TMPDIR"] = ENV["JOB_TMPDIR"]
        end
    end
end
```

## Accessing LEGEND Data 

Add the following in your `~/.bashrc` to use the most up-to-date data production: 
``` bash 
export LEGEND_DATA_CONFIG="/ptmp/lschl/l200/current/config.json"
```
This tells `LEGENDDataManagement.jl` where to find the data for the *current* production. 

To use VSCode with the julia extension, it is also necessary to add the following to your VSCode settings; `settings.json`: 
``` json
"terminal.integrated.env.linux": {
            "LEGEND_DATA_CONFIG": "/ptmp/lschl/l200/current/config.json"
        },
```

## Give mppgedet group acess to a directory: 
```
chown -R USER:mppgedet some_dir
chmod -R g+rwxs some_dir
setfacl -R -d -m g:mppgedet:rwx some_dir`

```
