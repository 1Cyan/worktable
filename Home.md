Welcome to the HBT2 wiki!

# Table Of Contents
* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Compile](#compile)
* [Run](#run)
* [Output](#output)
    - [basic data selection](#basic-data-selection)
* [Difference from `HBT-1` and `SUBFIND`](#notes-for-users-migrating-from-hbt-to-hbt2)
* [Reference](#reference)

## Introduction
HBT2 is a hybrid subhalo finder and merger tree builder for cosmological simulations. 

It comes with two editions:
* a [MPI](https://github.com/Kambrian/HBT2/tree/MPI-Hydro) edition that can be run on distributed clusters or shared memory machines. It is MPI/OpenMP parallelized.
* a [OpenMP](https://github.com/Kambrian/HBT2/tree/hydro) edition that can be run on shared memory machines. It is only OpenMP parallelized. This version is more memory efficient than the MPI branch on shared memory machines, and is more suitable for analysing zoomed-in simulations that are difficult to balance on distributed clusters.

Both editions support hydro simulations with gas/stars.

## Prerequisites

- a `c++` compiler with `c++11` support (e.g., `gcc 4.8.1` above)
- [HDF5](https://www.hdfgroup.org/) C library (1.8.0 and above)

For the MPI edition, you also need
- a MPI library (e.g., OpenMPI, MPICH, PlatformMPI)

### Optional dependence
- GNU Scientific Library [(GSL)](http://www.gnu.org/software/gsl/). 
Only needed if you want to output the shapes and orientations of subhaloes. To enable or disable GSL support, uncomment or comment out the GSL block in `Makefile.inc` (especially the `-DHAS_GSL` line). When enabled, HBT will do eigenvalue decomposition of the inertial tensor of each subhalo, and output the eigenvalue and eigenvectors describing the shape and direction of the subhalo. Without GSL, only the inertial tensors will be output.
- For the OpenMP edition, if you will be reading EAGLE/APOSTLE hdf5 data in parallel, you also need HDF5 compiled with multi-thread support.

## Compile
To produce single-precision `HBT` (internal datatypes are 4byte int and 4byte float), do

	make

. If you need double-precision, do

    make HBTdouble
    
### Customize the compilation

  You can enable some macro definitions to customize the behaviour of the code. These include:

- `HBT_INT8` : use 8 byte integer (i.e., C long int) as default integer type (for particle IDs etc.)
- `HBT_REAL8`: use 8 byte float (i.e., C double) as default float type (for particle position/velocity)
- `UNBIND_WITH_THERMAL_ENERGY`: include thermal energy in unbinding (only relevant for hydro simulations). If this is not defined, the code does not read in or use the thermal energy at all.
- `DM_ONLY` : compile the code for dark matter only simulations, or only process the DM particles while disregarding other types of particles. You do not have to turn this on for DM-only simulations, but doing this saves some memory (without DM_ONLY, HBT records one mass for each particle). 
     * For the openmp edition, enabling this flag assumes DM particles have the same mass. If this is not the case, do not enable this flag. 
     * For the MPI version, it's ok even if the DM particles have individual mass and this flag can always be used if you want to process only the DM particles.

  Simply add these macro definitions to the CXXFLAGS of your target in the Makefile. For example, adding this line to the Makefile
  
          HBT: CXXFLAGS+=-DHBT_INT8 

  will define `HBT_INT8` when you compile `HBT`.

## Run
### FoF halos
HBT needs simulation snapshots and halo catalogues (e.g., fof halos) as input. If you do not already have halo catalogues for your simulation, you can use the `FoF` program in HBT as detailed [here](https://github.com/Kambrian/HBTplus/wiki/FoF-halo-finder). You have to run it before running the main program of HBT.

### Subhalos and merger trees
The main function of `HBT` is to produce subhalos and their evolution histories. 

For the Hydro edition:
 
    ./HBT configs/Example.conf [snapshotstart] [snapshotend]

For the MPI edition:

    mpirun -np 2 ./HBT configs/Example.conf [snapshotstart] [snapshotend]

Check `configs/Example.conf` for a sample parameter file.

If `snapshotend` is omitted, only process `snapshotstart`. If `snapshotstart` is also omitted, will run from `MinSnapshotIndex` (default=0) to `MaxSnapshotIndex` (specified in config file). If `snapshotstart>MinSnapshotIndex`, `HBT` assumes the previous run stopped at `snapshotstart-1`, and will load the `SrcSnap_($snapshotstart-1).hdf5` and continue the run from `snapshotstart`. 

To submit to a batch queue, check `HBTjob.bsub`

### EAGLE runs
To process EAGLE outputs, you will have to create a snapshotlist.txt file (due to the complex snapshot names) and place it under the subhalo path. There is a script toolbox/CreateSnapshotlist.py for creating the list.

## Output
The outputs are in HDF5 format, which can be viewed with [HDFView](https://www.hdfgroup.org/products/java/hdfview/index.html) or any other HDF tools. In python, you can use [h5py](https://pypi.python.org/pypi/h5py) to read them directly. A python reader module is also provided in `toolbox/HBTReader.py`. There are two types of files in the output:
  
- SubSnap_*.hdf5: the subhalo catalogues, where * is replaced by the snapshot index (normally from `0` to `NumberOfSnapshots-1`). The snapshot index may not be the same as the `SnapshotId` which is the numbering of the simulation snapshot, depending on whether the `SnapshotId`s are consecutive and whether a `SnapshotList` parameter is specified in the config file. Note that if a `MinSnapshotIndex` is specified in the config file, the outputs will start from `MinSnapshotIndex`. This is useful when there are no objects in the first few snapshots of the simulation so that they can be skipped.
- SrcSnap_*.hdf5: source subhalo catalogues, only used for restarting HBT from an intermediate snapshot. Can be safely removed once the run has finished.

Besides, the `Parameters.log` file records the version number of HBT used, as well as the parameter values used.

Each subhalo is labelled by a unique `TrackId`, which is fixed throughout its evolution history (see illustration in the figure). So doing merger tree with HBT is straightforward: the progenitor/descendent of a subhalo at another snapshot is simply the subhalo labelled by the same `TrackId` at that time. The host halo of each subhalo is given by `HostId`, which is the index of the host halo in the order stored in the corresponding (FoF) halo catalogue. To facilitate fast retrieval of all the subhaloes in each host halo, the `/Membership/GroupedTrackIds` dataset in the file stores the list of subhaloes in each group (Note this is only available for the OpenMP version of HBT2).  

![TrackTable](https://github.com/Kambrian/HBTplus/blob/doc/tracktable.png)

`Rank` gives the order of subhaloes inside the group if sorted according to `Nbound`, with `Rank=0` indicating the most-massive subhalo inside each group (i.e., the main/central subhalo).

`Depth` gives the level of the subhalo in the merging hierarchy. A central subhalo has `Depth=0`; those directly merged to the host halo of the central have `Depth=1` (i.e., sub-subhalos); those directly merged to depth=1 subhalos have `Depth=2` (i.e., sub-sub-subhalos).

`Nbound` gives the number of bound particles in the subhalo. Once a subhalo is stripped to below `MinNumPartOfSub` specified in the parameter file, HBT continues to track its most bound particle. This single-particle descendent then has `Nbound=1`, and represent the "orphan" galaxy population in the framework of semi-analytical models. These orphans are also listed as subhaloes. `Mbound` is the bound mass (in physical units). Correspondingly, `NboundType` and `MboundType` are the bound particle number and bound mass for each type of particles (e.g., gas, DM, star, boundary..., relevant only for hydro simulations). By default, the mass of a subhalo does not include the contribution from its sub-subhalos (similar to the mass definition in `SUBFIND`).

`MVir`, `RVirComoving`, etc are the virial mass and radius for each bound subhalo, obtained by searching for a spherical overdensity (SO) radius counting only the bound density. This could differ slightly from the SO quantities for the host halo defined using all the mass (no matter bound or not) enclosed in a sphere. At low redshift, the 200Mean mass can be underestimated by 10%. However, the 200Crit and the tophat virial masses are generally unbiased since almost all the masses inside these two radii are found in the bound structure of the FoF halo. 

To obtain proper spehrical overdensity quantities for the host halo, please compile and run `halo_virial.cpp` after HBT finishes:

    cd toolbox
    make halo_virial
    ./halo_virial [config_file] [snapshot_number]

Note that this program is only available in the [Hydro](https://github.com/Kambrian/HBTplus/tree/Hydro) branch of the code, which is to be run on a shared memory machine. The `Hydro` branch is aware of the output format of the `MPI-Hydro` branch, so that you can run the `MPI-Hydro` version of `HBT`, and the `Hydro` version of `halo_virial`.

`SinkTrackId`: the `TrackId` that this subhalo merged to; `-1` if this subhalo has not merged to any other track. A subhalo merger is defined when two subhalos are nearly identical in location and velocities given the current resolution of the simulation. When this is detected, the `SinkTrackId` of the satellite subhalo is set to the `TrackId` of its host-subhalo (the other one in the pair; whether a subhalo is a satellite or a host is determined according to the current `Rank` and `Depth`). The satellite is then merged to its host-subhalo (i.e., transferring all its particles to the host-subhalo) unless `MergeTrappedSubhalos` is set to `0` in the config file.   


The other properties should be self-explainatory. 

Notes on Peebles and Bullock spin parameters: these parameters are vaguely defined due to the ambiguity/lack of standard in the mass, radius, and energy of a subhalo. So we do not provide them in our output. If you really need the spins, you can still compute them easily from the relevant quantities (mass, radius, energy, angular momentum) as you feel appropriate. If possible, use the `SpecificAngularMomentum` instead of the spin parameters. 

There might be objects with `Nbound=0` and an empty particle list. These are mostly eliminated tracks arising from small halos that had their most-bound particles fluctuated away from the halo itself and then back again, creating duplicate branches which are eliminated later. In hydro simulations, `Nbound=0` tracks could also exist as a result of all its particles consumed by a black hole. 

### Basic Data Selection

For scientific analysis of the tracks, we recommend a basic selection in `LastMaxMass` (the peak mass of each track), to eliminate under-resolved tracks.  

## Notes for users migrating from `HBT` to `HBT2`
HBT and HBT2 have different algorithmic details. They are not expected to give identical results. 

HBT no longer uses `ProSubID`. Instead, each subhalo is labelled by a unique `TrackId`, which is fixed throughout its evolution history. The progenitor/descendent of a subhalo at another snapshot is simply the subhalo labelled by the same `TrackId` at that time. 

`sub_hierarchy` is not available in HBT2. Instead, a list of `NestedSubhalos` is available for each subhalo.

The host halo of each subhalo is given by `HostHaloId`, which is the index of the host halo in the order stored in the corresponding (FoF) halo catalogue.  With this you can sort or search to find all the members of each host.

HBT2 no longer have splintters. HBT2 does not store fake haloes either, i.e., for haloes that are not bound, you won't be able to find any subhalo hosted by it in HBT2.

## Reference
For now, please cite the original [HBT paper](http://adsabs.harvard.edu/abs/2012MNRAS.427.2437H) if you use HBT in your work. We will soon have another paper coming out describing the new implementation here.

