### Output files
There are two types of files in the output:
  
- SubSnap_*.hdf5: the subhalo catalogues, where * is replaced by the snapshot index (normally from `0` to `NumberOfSnapshots-1`). The snapshot index may not be the same as the `SnapshotId` which is the numbering of the simulation snapshot, depending on whether the `SnapshotId`s are consecutive and whether a `SnapshotList` parameter is specified in the config file. Note that if a `MinSnapshotIndex` is specified in the config file, the outputs will start from `MinSnapshotIndex`. This is useful when there are no objects in the first few snapshots of the simulation so that they can be skipped.
- SrcSnap_*.hdf5: source subhalo catalogues, only used for restarting HBT from an intermediate snapshot. Can be safely removed once the run has finished.

Besides, the `Parameters.log` file records the version number of HBT used, as well as the parameter values used.

The main outputs are in HDF5 format, which can be viewed with [HDFView](https://www.hdfgroup.org/products/java/hdfview/index.html) or any other HDF tools. In python, you can use [h5py](https://pypi.python.org/pypi/h5py) to read them directly. A python reader module is also provided in `toolbox/HBTReader.py`. 

The following is a screenshot of a sample SubSnap file opened in hdfview:
![Sample SubSnap file in hdfview](https://github.com/Kambrian/HBTplus/blob/doc/SubSnap.png)

### Subhalo properties and merger tree
Each subhalo is labelled by a unique `TrackId`, which is fixed throughout its evolution history (see illustration in the figure). So doing merger tree with HBT is straightforward: the progenitor/descendent of a subhalo at another snapshot is simply the subhalo labelled by the same `TrackId` at that time. The host halo of each subhalo is given by `HostHaloId`, which is the index of the host halo in the order stored in the corresponding (FoF) halo catalogue (those in the first FoF halo have `HostHaloId=0`). There could be subhalos with `HostHaloId=-1`. These are subhalos that are not enclosed in any FoF halo but still remain bound. To facilitate fast retrieval of all the subhaloes in each host halo, the `/Membership/GroupedTrackIds` dataset in the file stores the list of subhaloes in each group (Note this is only available for the OpenMP version of HBT2).  

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

`SinkTrackId`: the `TrackId` that this subhalo merged to; `-1` if this subhalo has not merged to any other track. A subhalo merger is defined when two subhalos are nearly identical in location and velocities given the current resolution of the simulation. When this is detected, the `SinkTrackId` of the satellite subhalo is set to the `TrackId` of its host-subhalo (the other one in the pair; whether a subhalo is a satellite or a host is determined according to the current `Rank` and `Depth`). The satellite is then merged to its host-subhalo (i.e., transferring all its particles to the host-subhalo) unless `MergeTrappedSubhalos` is set to `0` in the config file.   


The other properties are hopefully self-explainatory. All the subhalo fields are defined in `src/subhalo.h` which may contain more comments about each.

Notes on Peebles and Bullock spin parameters: these parameters are vaguely defined due to the ambiguity/lack of standard in the mass, radius, and energy of a subhalo. So we do not provide them in our output. If you really need the spins, you can still compute them easily from the relevant quantities (mass, radius, energy, angular momentum) as you feel appropriate. If possible, use the `SpecificAngularMomentum` instead of the spin parameters. 

There might be objects with `Nbound=0` and an empty particle list. These are mostly eliminated tracks arising from small halos that had their most-bound particles fluctuated away from the halo itself and then back again, creating duplicate branches which are eliminated later. In hydro simulations, `Nbound=0` tracks could also exist as a result of all its particles consumed by a black hole. 

### Basic Data Selection

For scientific analysis of the tracks, we recommend a basic selection in `LastMaxMass` (the peak mass of each track), to eliminate under-resolved tracks.  

--------------------
Here's the headers of an example output file.

Special types: 
- `'O'`: Variable length array. 

  It's an array of array, the subarrays have variable shape.

- `'V'`: Structure array. 

  `'V28'` means that one item of the array contains 28 bytes.

  `fp['Subhalos'][0]['TrackId']` and `fp['Subhalos']['TrackId'][0]` are identical.

```
SubSnap_059.hdf5
Grp /
Grp /Cosmology/
Dat /Cosmology/HubbleParam 	   f4 (1,)
Dat /Cosmology/OmegaLambda0 	   f4 (1,)
Dat /Cosmology/OmegaM0 	   f4 (1,)
Dat /Cosmology/ParticleMass 	   f4 (1,)
Dat /Cosmology/ScaleFactor 	   f4 (1,)
Grp /Membership/
Dat /Membership/GroupedTrackIds 	    O (523155,)
    /Membership/GroupedTrackIds[0] 	   i4 (4438,)
    ...
Dat /Membership/NumberOfFakeHalos 	   i4 (1,)
Dat /Membership/NumberOfNewSubhalos 	   i4 (1,)
Dat /NestedSubhalos 	    O (632298,)
    /NestedSubhalos[0] 	   i4 (0,)
    ...
Dat /ParticleProperties 	    O (632298,)
    /ParticleProperties[0] 	  V28 (1,)
      'ParticleIndex' 	   i4
      'ComovingPosition' 	   f4 (3,)
      'PhysicalVelocity' 	   f4 (3,)
    ...
Dat /SnapshotId 	   i4 (1,)
Dat /SubhaloParticles 	    O (632298,)
    /SubhaloParticles[0] 	   i4 (1,)
    ...
Dat /Subhalos 	 V300 (632298,)
      'TrackId' 	   i4
      'Nbound' 	   i4
      'Mbound' 	   f4
      'HostHaloId' 	   i4
      'Rank' 	   i4
      'LastMaxMass' 	   f4
      'SnapshotIndexOfLastMaxMass' 	   i4
      'SnapshotIndexOfLastIsolation' 	   i4
      'SnapshotIndexOfBirth' 	   i4
      'SnapshotIndexOfDeath' 	   i4
      'RmaxComoving' 	   f4
      'VmaxPhysical' 	   f4
      'LastMaxVmaxPhysical' 	   f4
      'SnapshotIndexOfLastMaxVmax' 	   i4
      'R2SigmaComoving' 	   f4
      'RHalfComoving' 	   f4
      'R200CritComoving' 	   f4
      'R200MeanComoving' 	   f4
      'RVirComoving' 	   f4
      'M200Crit' 	   f4
      'M200Mean' 	   f4
      'MVir' 	   f4
      'SpecificSelfPotentialEnergy' 	   f4
      'SpecificSelfKineticEnergy' 	   f4
      'SpecificAngularMomentum' 	   f4 (3,)
      'SpinPeebles' 	   f4 (3,)
      'SpinBullock' 	   f4 (3,)
      'InertialEigenVector' 	   f4 (3, 3)
      'InertialEigenVectorWeighted' 	   f4 (3, 3)
      'InertialTensor' 	   f4 (6,)
      'InertialTensorWeighted' 	   f4 (6,)
      'ComovingAveragePosition' 	   f4 (3,)
      'PhysicalAverageVelocity' 	   f4 (3,)
      'ComovingMostBoundPosition' 	   f4 (3,)
      'PhysicalMostBoundVelocity' 	   f4 (3,)
```