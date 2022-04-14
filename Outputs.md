# Table of cotents
* [Output files](#output-files)
* [Subhalos and merger tree](#subhalo-properties-and-merger-tree)
* [Examples for using `HBTReader`](#examples-for-using-hbtreader)
* [Basic data selection](#basic-data-selection)
* [Potential Complications](#potential-complications)
* [Difference from `HBT-1`](#notes-for-users-migrating-from-hbt-to-hbt)
* [Grouping trees into disconnected forests](#grouping-trees-into-forests)

### Output files
There are two types of files in the output:
  
- SubSnap_*.hdf5: the subhalo catalogues, where * is replaced by the snapshot index (normally from `0` to `NumberOfSnapshots-1`). The snapshot index may not be the same as the `SnapshotId` which is the numbering of the simulation snapshot, depending on whether the `SnapshotId`s are consecutive and whether a `SnapshotList` parameter is specified in the config file. Note that if a `MinSnapshotIndex` is specified in the config file, the outputs will start from `MinSnapshotIndex`. This is useful when there are no objects in the first few snapshots of the simulation so that they can be skipped.
- SrcSnap_*.hdf5: source subhalo catalogues, only used for restarting HBT from an intermediate snapshot. Can be safely removed once the run has finished. When restarting HBT from `snapnum`, only the last snapshot, `SrcSnap_[snapnum-1].hdf5`, is loaded, so actually all the previous SrcSnap except the last can be removed.

These outputs are in HDF5 format, which can be viewed with [HDFView](https://www.hdfgroup.org/products/java/hdfview/index.html) or any other HDF tools. 
- In python, you can use [h5py](https://pypi.python.org/pypi/h5py) to read them directly. 
- A python reader module is also provided in `toolbox/HBTReader.py`. 
- To analyze them in C/C++, the simple way is to take one of the postprocessing programs under `toolbox` folder as a template and modify it. 
- The data fields of subhalos are defined in `src/subhalo.h` and the output function is defined in `src/io/subhalo_io.cpp`, where the HDF5 datatype for subhalo is also created. The subhalo properties are saved using a single compound hdf5 datatype, and the subhalo particles are saved using variable length hdf5 arrays.

The following is a screenshot of a sample SubSnap file opened in hdfview:
![Sample SubSnap file in hdfview](https://github.com/Kambrian/HBTplus/blob/doc/SubSnap.png)

Besides, the `Parameters.log` file records the version number of HBT+ used, as well as the parameter values used.

### Subhalo properties and merger tree
Each subhalo is labelled by a unique `TrackId`, which is fixed throughout its evolution history (see illustration in the figure). So doing merger tree with HBT is straightforward: the progenitor/descendent of a subhalo at another snapshot is simply the subhalo labelled by the same `TrackId` at that time. The host halo of each subhalo is given by `HostHaloId`, which is the index of the host halo in the order stored in the corresponding (FoF) halo catalogue (those in the first FoF halo have `HostHaloId=0`). There could be subhalos with `HostHaloId=-1`. These are subhalos that are not enclosed in any FoF halo but still remain bound. To facilitate fast retrieval of all the subhaloes in each host halo, the `/Membership/GroupedTrackIds` dataset in the file stores the list of subhaloes in each group (Note this is only available for the OpenMP version of HBT+).  

![TrackTable](https://github.com/Kambrian/HBTplus/blob/doc/tracktable.png)

`Rank` gives the order of subhaloes inside the group if sorted according to `Nbound`, with `Rank=0` indicating the most-massive subhalo inside each group (i.e., the main/central subhalo).

`Depth` gives the level of the subhalo in the merging hierarchy. A central subhalo has `Depth=0`; those directly merged to the host halo of the central have `Depth=1` (i.e., sub-subhalos); those directly merged to depth=1 subhalos have `Depth=2` (i.e., sub-sub-subhalos).

`Nbound` gives the number of bound particles in the subhalo. Once a subhalo is stripped to below `MinNumPartOfSub` specified in the parameter file, HBT continues to track its most bound particle. This single-particle descendent then has `Nbound=1`, and represent the "orphan" galaxy population in the framework of semi-analytical models. These orphans are also listed as subhaloes. `Mbound` is the bound mass (in physical units). Correspondingly, `NboundType` and `MboundType` are the bound particle number and bound mass for each type of particles (e.g., gas, DM, star, boundary..., relevant only for hydro simulations). By default, the mass of a subhalo does not include the contribution from its sub-subhalos (similar to the mass definition in `SUBFIND`).

`MVir`, `RVirComoving`, etc are the virial mass and radius for each bound subhalo, obtained by searching for a spherical overdensity (SO) radius counting only the bound density. This could differ slightly from the SO quantities for the host halo defined using all the mass (no matter bound or not) enclosed in a sphere. At low redshift, the 200Mean mass can be underestimated by 10%. However, the 200Crit and the tophat virial masses are generally unbiased since almost all the masses inside these two radii are found in the bound structure of the FoF halo. 

To obtain proper spherical overdensity quantities for the host halo, please compile and run `halo_virial.cpp` after HBT finishes:

    cd toolbox
    make halo_virial
    ./halo_virial [config_file] [snapshot_number]

Note that this program is only available in the [Hydro](https://github.com/Kambrian/HBTplus/tree/Hydro) branch of the code, which is to be run on a shared memory machine. The `Hydro` branch is aware of the output format of the `MPI-Hydro` branch, so that you can run the `MPI-Hydro` version of `HBT`, and the `Hydro` version of `halo_virial`.

`SinkTrackId`: the `TrackId` that this subhalo merged to; `-1` if this subhalo has not merged to any other track. A subhalo merger is defined when two subhalos are nearly identical in location and velocities given the current resolution of the simulation. When this is detected, the `SinkTrackId` of the satellite subhalo is set to the `TrackId` of its host-subhalo (the other one in the pair; whether a subhalo is a satellite or a host is determined according to the current `Rank` and `Depth`). The satellite is then merged to its host-subhalo (i.e., transferring all its particles to the host-subhalo) unless `MergeTrappedSubhalos` is set to `0` in the config file.  Such "sink" events are detected for orphan/disrupted (aka `Nbound=1`) subhalos as well, so that one may rely on the `SinkTrackId` to know where the mostbound particle in the orphan track have merged to. Note the most bound particle is also merged to the host track, so that duplicate particles could be found after the merger between the host track and the sunk track.  


The other properties are hopefully self-explainatory. All the subhalo fields are defined in `src/subhalo.h` which may contain more comments about each.

- Notes on Peebles and Bullock spin parameters: 

    These parameters are vaguely defined due to the ambiguity/lack of standard in the mass, radius, and energy of a subhalo. So we do not provide them in our output. If you really need the spins, you can still compute them easily from the relevant quantities (mass, radius, energy, angular momentum) as you feel appropriate. If possible, use the `SpecificAngularMomentum` instead of the spin parameters. 

### Examples for using `HBTReader`
In order to tell python where to find HBTReader, you need to add its path to your environment variable `PYTHONPATH`. In `bash`, you can do

    export PYTHONPATH=/path/to/hbtreader:$PATHONPATH

replace `/path/to/hbtreader` with the actual path of `HBTReader.py`, which is the `toolbox` directory of your downloaded source.

After that you can open your favorite python console (for example, `ipython`) and use the reader. Suppose your HBT+ output is located at `/hbt/output`, then inside your python console:

     from HBTReader import HBTReader
     reader=HBTReader('/hbt/output')

The reader will parse the parameter file located in the output folder and initialize the reader accordingly.

To load a given subhalo snapshot `snapshotnumber`, you can do:

     subs=reader.LoadSubhalos(snapshotnumber) #load all

You can use `-1` as the snapshotnumber to represent the last snapshot in the above function. Negative numbers means counting from the lastsnapshot backwards. This will load all the subhalos at `snapshotnumber` into a numpy struct array. After loading, the subhalos are at your hand to analyze. For example

     subs['Nbound']

gives the number of bound particles in each subhalo.
     
     subs['LastMaxMass'][subs['Nbound']>1000]

gives you the peak mass of all subhalos with more than 1000 particles. You can also sort easily:

     subs.sort(order='TrackId') #sort using trackid
     subs.sort(order=['HostHaloId', 'Rank']) # sort using hosthaloid first, and then using 'Rank' inside the same hosthalo

To see all the properties available, simply show the datatype of the array:

     subs.dtype

This will show something like:

      Out[30]: dtype([('TrackId', '<i8'), ('Nbound', '<i8'), ('Mbound', '<f4'), ('HostHaloId', '<i8'), ('Rank', '<i8'), ('Depth', '<i4'), ('LastMaxMass', '<f4'), ('SnapshotIndexOfLastMaxMass', '<i4'), ('SnapshotIndexOfLastIsolation', '<i4'), ('SnapshotIndexOfBirth', '<i4'), ('SnapshotIndexOfDeath', '<i4'), ('SnapshotIndexOfSink', '<i4'), ('RmaxComoving', '<f4'), ('VmaxPhysical', '<f4'), ('LastMaxVmaxPhysical', '<f4'), ('SnapshotIndexOfLastMaxVmax', '<i4'), ('R2SigmaComoving', '<f4'), ('RHalfComoving', '<f4'), ('BoundR200CritComoving', '<f4'), ('BoundM200Crit', '<f4'), ('SpecificSelfPotentialEnergy', '<f4'), ('SpecificSelfKineticEnergy', '<f4'), ('SpecificAngularMomentum', '<f4', (3,)), ('InertialTensor', '<f4', (6,)), ('InertialTensorWeighted', '<f4', (6,)), ('ComovingAveragePosition', '<f4', (3,)), ('PhysicalAverageVelocity', '<f4', (3,)), ('ComovingMostBoundPosition', '<f4', (3,)), ('PhysicalMostBoundVelocity', '<f4', (3,)), ('MostBoundParticleId', '<i8'), ('SinkTrackId', '<i8')])


In some cases you might only want to load specified fields or objects to speed up the io:

     nbound=reader.LoadSubhalos(snapshotnumber, 'Nbound') #only Nbound
     sub2=reader.LoadSubhalos(snapshotnumber, subindex=2) #only subhalo 2

You can also load the entire history of a given track:

     track2=reader.GetTrack(2) #track 2

Check the `HBTReader.py` for more functions.

### Basic Data Selection

For scientific analysis of the tracks, we recommend a basic selection in `LastMaxMass` (the peak mass of each track), to eliminate under-resolved tracks.  

### Potential complications

- Empty tracks

    There might be objects with `Nbound=0` and an empty particle list. These are mostly eliminated tracks arising from small halos that had their most-bound particles fluctuated away from the halo itself and then back again, creating duplicate branches which are eliminated later. Fragmentations in the FoF at the infall snapshot could also lead to the creation of duplicate branches that are subsequently eliminated. These tracks are typically with a very low peak mass (`LastMaxMass<50 particles`) and live for a very short period (`SnapshotIndexOfDeath-SnapshotIndexOfBirth<~3`), so they can be safely ignored. 

    In hydro simulations, `Nbound=0` tracks could also exist as a result of all its particles consumed by a black hole. 

- Duplicate particles
   
    Although HBT+ subhalos are defined to have exclusive particle lists, a trace amount of duplicate particles can be expected after subhalos switch hosts. This is because the source subhalos, from which the self-bound subhalos are computed, are inherited from progenitors that share some particles with their hosts, to allow for accretion within the subgroups. 

## Notes for users migrating from `HBT` to `HBT+`
HBT and HBT+ have different algorithmic details. They are not expected to give identical results. 

HBT+ no longer uses `ProSubID`. Instead, each subhalo is labelled by a unique `TrackId`, which is fixed throughout its evolution history. The progenitor/descendent of a subhalo at another snapshot is simply the subhalo labelled by the same `TrackId` at that time. 

`sub_hierarchy` is not available in HBT+. Instead, a list of `NestedSubhalos` is available for each subhalo.

The host halo of each subhalo is given by `HostHaloId`, which is the index of the host halo in the order stored in the corresponding (FoF) halo catalogue.  With this you can sort or search to find all the members of each host.

HBT+ no longer have splintters. HBT+ does not store fake haloes either, i.e., for haloes that are not bound, you won't be able to find any subhalo hosted by it in HBT+.

## Grouping trees into forests
Some analysis might want to group the tracks into forests of connected tracks that can be processed independently from other forests. A module [Tree2Forest.py](https://github.com/Kambrian/HBTplus/blob/Hydro/toolbox/Track2Forest.py) has been added in the toolbox in the [Hydro branch](https://github.com/Kambrian/HBTplus/tree/Hydro) to accomplish this.