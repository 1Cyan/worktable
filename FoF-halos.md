HBT needs simulation snapshots and halo catalogues (e.g., fof halos) as input. If you do not already have halo catalogues for your simulation, we have provided a `FoF` code to compute them. 

Currently only a serial version is provided in the `Hydro` branch, and it works on a single type of particle (i.e., dark matter). 

# Compile and run
To use it,

    make FoF
    ./FoF [config_file] <snapshot_index_start> <snapshot_index_end>

- Examples:
    * process all the snapshots (from `MinSnapshotIndex` to `MaxSnapshotIndex`)
    
            ./FoF configs/Example.conf

    * process snapshot 10:

            ./FoF configs/Example.conf 10

    * run from snapshot 5 to 20 (both end included):

            ./FoF configs/Example.conf 5 20

# Use the output
To use these FoF halos as input to `HBT`, set

    GroupFileFormat hbt_group_hdf5

in the config file.

# Customization
You can also change the following parameters of FoF:

    FoFLinkParam 0.2  #fof linking length in unit of average particle separation
    FoFSizeMin 20  #minimum number of particles in halo
    FoFDarkMatterMassFraction 1.0 #OmegaDM0/OmegaMatter0, the mass fraction of dark matter in the simulation, used to determine average particle separation from particle mass. If there is no baryons, then set this to 1.

# Computing Halo properties

The virial mass/radius and `Rmax`/`Vmax` quantities for FoF halos (**not** the subhalos, the properties of which are computed inside `HBT`) can be computed using `halo_virial.cpp` in toolbox of the `Hydro` branch. To compute these quantities, 

    cd toolbox
    make halo_virial
    ./halo_virial [config_file] [snapshot_number]

Note this has to be run **after finding subhalos with `HBT` **, because we use the center of the central subhalo as the center for the host halo in computing host properties.

Remember to change the datatype by defining/undefining [`HBT_INT8` and `HBT_REAL8`](https://github.com/Kambrian/HBTplus/wiki#customize-the-compilation) in the makefile before compiling.