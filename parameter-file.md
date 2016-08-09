You can find a few parameter files under config/. Below is the content of Example.conf:

	#sample config file. Case sensitive. anything after a "#" or "[" are comments and ignored by parser.

	[Compulsary Params]
	SnapshotPath  /path/to/your/snapshot/files
	HaloPath /path/to/your/group/files
	SubhaloPath /path/to/output/your/subhaloes
	SnapshotFileBase snap_milli                #prefix to snapshot file name
	MaxSnapshotIndex 63                        #Index of final snapshot
	BoxSize 62.5                               #Boxsize, for consistency check
	SofteningHalo 5e-3                         #softening of dark matter particles

	[Optional]
	#the default values for optional parameters are listed below, modify and uncomment each item to enable

	[Reader]
	#SnapshotFormat gadget

	#GroupFileFormat gadget3_int #other format: gadget3_long, gadget2_int, gadget2_long, where long/int specifies the integer datatype in the file

	#MaxConcurrentIO 4

	[Units]
	#MassInMsunh 1e10
	#LengthInMpch 1
	#VelInKmS 1

	#MinNumPartOfSub 20

	#SaveSubParticleProperties 1               #save position and velocity for each subhalo particle; set to 0 to disable.

	[Performance]
	#MaxSampleSizeOfPotentialEstimate 1000     #the maximum number of particles to use when calculating potential for unbinding purpose. A value of 1000 should lead to percent level accuracy in bound mass. Larger values leads to more accurate unbinding, but would be slower. Set to 0 to use all particles available, i.e., to disable sampling and calculate binding energy precisely.