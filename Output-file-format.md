Outputs' format is hdf5.
![example SubSnap file](https://drive.google.com/file/d/0B049M2pH2eETWV9SRnRYMWt6bTg/view?usp=sharing)

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