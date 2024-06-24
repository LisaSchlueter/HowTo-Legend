# Juleana setup at NERSC
## SSH connection
1. Get NERSC account
2. Create [MFA](https://docs.nersc.gov/connect/mfa/) (multi-factor authetification) to get OPT (one-time-password):
    * Once you have the account, you can login to nersc:
	   `ssh <username>@perlmutter.nersc.gov`   
       replace `<username>` with your nersc username.
3. Download [sshproxy](https://docs.nersc.gov/connect/mfa/#mfa-for-ssh-keys-sshproxy)
    * This creates ssh key that is valid 24 hours. No need to retype password+OTP everytime
	* usage: `./sshproxy.sh -u <username>`    
		
4. Save connection in `~/.ssh/config` config to make connecting easier. Then login via `ssh perlmutter`
```
Host perlmutter
    HostName perlmutter.nersc.gov
    User <username>
    IdentityFile ~/.ssh/nersc
    IdentitiesOnly yes
    ForwardAgent yes
```

## Software on NERSC

- `module avail` show list of globally installed software 
- `module load <module-name>` load module to session
- `module show <module-name>` show information about module (PATH,...)
---

## Julia on NERSC

- `module load julia` load default julia version
or to get specific version, e.g. julia 1.10.2: 
- `/global/common/software/nersc/n9/julia/1.10.2/bin/julia` 

## VS-Code on NERSC (on a login node)
Open VS-Code on your computer as usual. Connect to NERSC via `Connect to Host`. Once logged-in, several julia settings that have to me modified. Go to `settings -> remote [SSH: permutter]-> extensions -> julia` 
1. Set environmental path to some directory in home directory. As an example my case for me (Lisa Schlueter):
```
"julia.environmentPath": "/global/homes/l/lschl/julia-envs/legend-dev" 
```
2. set executable path to julia version, that you want to use. for example:
```
"julia.executablePath": "/global/common/software/nersc/n9/julia/1.10.2/bin/julia" 
 ```

## Add Julia-Legend registry
To be able to load Juleana modules run: 
```
julia> using Pkg; pkg"registry add General https://github.com/legend-exp/LegendJuliaRegistry.git"
```
This needs to be done only once. For reference see [confluence](https://legend-exp.atlassian.net/wiki/spaces/LEGEND/pages/494632973/Julia+Software+Stack#The-LEGEND-Julia-package-registry).
