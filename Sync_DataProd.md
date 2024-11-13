# Synchronize directories from data production between servers
This file describes how to synchronize directories from Server A to Server B. This is useful, for example, in the case when the juleana data-production is conducted on server A (e.g. raven at MPCDF), but the code development happens on server B (e.g. perlmutter at NERSC).  
## Directory structure
The Juleana data production direcory, here called `current` has this sturcture:
```
current/
    - jlenv/ 
    - legend-metadata/
    - generated/
```
with 
* `jlenv` containing a `Project.toml` and `Manifest.toml` that define the julia environment used for the data production. 
* `legend-metadata` clone of [git repo](https://github.com/legend-exp/legend-metadata)
* `generated` directory with juleana results `.dsp`, `.hitch`, `.par`...
The generated directory has the following structure:
```
generated/
    - jllog/
    - jlplt/
    - jlreport/
    - jlpar/
        - p03/
            - r000/
            - r001/
        - p04/
        - ...
    - tier/
        - jldsp/
        - jlhitch/
        - jlpulsch/
        - ...
```
## Sychronization with rsync
`rsync` is an easy way to copy data from one place to another. The advantage over using `scp` is that file that didn't change are not copied again. To synchronize a directory from serverA to serverB run the following command
```
rsync -i -a -v --progress user@serverA:path-to-dir  user@serverB:path-to-dir
```
* `--progress` show progress
* `-i` itemize-changes
* `-v` more detailed information about the synchronization process
* `-a` archive options. This includes: 
    * `-r` Recursive (transfers directories and their contents recursively).
    * `-l` Links (copies symlinks as symlinks).
    * `-p` Perms (preserves file permissions).
    * `-t` Times (preserves modification times).
    * `-g` Group (preserves group ownership).
    * `-o` Owner (preserves user ownership).
    * `-D` Devices (preserves device and special files).
## Example
In this example I (user = `lschl`) want to sychronize the `jlpar` directory from raven to perlmutter. I run the following command while being logged-in on raven: 
```
rsync -i -a -v --progress /ptmp/lschl/l200/current/generated/jlpar  lschl@perlmutter.nersc.gov:/global/cfs/projectdirs/m2676/data/lngs/l200/juleana/lschl/generated/
```
