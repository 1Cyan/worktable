Welcome to the HBT2 wiki!

# Table Of Contents
* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Compile](#compile)
* [Run](#run)
* [Reference](#reference)

## Introduction
HBT2 is a hybrid subhalo finder and merger tree builder for cosmological simulations. 

It comes with two editions:
* a [MPI](https://github.com/Kambrian/HBT2/tree/MPI-Hydro) edition that can be run on distributed clusters or shared memory machines. It is MPI/OpenMP parallelized.
* a [OpenMP](https://github.com/Kambrian/HBT2/tree/hydro) edition that can be run on shared memory machines. It is only OpenMP parallelized. This version is more memory efficient than the MPI branch on shared memory machines, and is more suitable for analysing zoomed-in simulations that are difficult to balance on distributed clusters.

Both editions support hydro simulations with gas/stars.

## Prerequisites

- a `c++` compiler with `c++11` support (e.g., `gcc 4.8.1` above)
  * if you use intel compilers, they still have to be built with a recent `gcc` to be able to support `c++11`. You can check the version of your compiler by `icc -V` or `gcc -v` for example.
  * if you have to install `gcc` from source, read the official installation guide from the gcc website and you will find that it's not as difficult. You can easily download all the prerequisites of gcc with a script provided in the source and you can compile in parallel (with `make -j`).
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
- `DM_ONLY` : compile the code for dark matter only simulations, or only process the DM particles while disregarding other types of particles. You do not have to turn this on for DM-only simulations, but doing this saves some memory (without DM_ONLY, HBT records one mass for each particle). 
     * For the openmp edition, enabling this flag assumes DM particles have the same mass. If this is not the case, do not enable this flag. 
     * For the MPI version, it's ok even if the DM particles have individual mass and this flag can always be used if you want to process only the DM particles.

- `SAVE_BINDING_ENERGY`: output the binding energy of each particle 

For unbinding gas particles:
- `UNBIND_WITH_THERMAL_ENERGY`: include thermal energy in unbinding (only relevant for hydro simulations). If this is not defined, the code does not read in or use the thermal energy at all.
- `HAS_THERMAL_ENERGY`: read in the thermal energy of each particle from the snapshot. Automatically defined if `UNBIND_WITH_THERMAL_ENERGY` is defined. If SaveSubParticleProperties is 1 in the config file, then the thermal energy will also be saved (so that you can use it to redefine the binding energy of each particle). 


  Simply add these macro definitions to the CXXFLAGS of your target in the Makefile. For example, adding this line to the Makefile
  
          HBT: CXXFLAGS+=-DHBT_INT8 

  will define `HBT_INT8` when you compile `HBT`.

## Run
### FoF halos
HBT needs simulation snapshots and halo catalogues (e.g., fof halos) as input. If you do not already have halo catalogues for your simulation, you can use the `FoF` program in HBT as detailed [here](https://github.com/Kambrian/HBTplus/wiki/FoF-halos). You have to run it before running the main program of HBT.

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

## Reference
For now, please cite the original [HBT paper](http://adsabs.harvard.edu/abs/2012MNRAS.427.2437H) if you use HBT in your work. We will soon have another paper coming out describing the new implementation here.

