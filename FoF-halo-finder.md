HBT needs simulation snapshots and halo catalogues (e.g., fof halos) as input. If you do not already have halo catalogues for your simulation, you can use the `FoF` program (currently only a serial version is provided in the `Hydro` branch). To use it,

    make FoF
    ./FoF configs/Example.conf snapshot_id

It will process one snapshot at a time. 

To use these FoF halos as input to `HBT`, set

    GroupFileFormat hbt_group_hdf5

in the config file.

You can also change the following parameters of FoF:

    FoFLinkParam 0.2  #fof linking length in unit of average particle separation
    FoFSizeMin 20  #minimum number of particles in halo
    FoFDarkMatterMassFraction 1.0 #OmegaDM0/OmegaMatter0, the mass fraction of dark matter in the simulation, used to determine average particle separation from particle mass. If there is no baryons, then set this to 1.