To read halo data in your own format, modify `src/io/custom_io.cpp` to add your own halo-reader code. The reader will be called in `src/io/halo_io.cpp` under `my_group_format`. To use this reader, set
 
    `GroupFileFormat my_group_format`
 
in config file.

The snapshot io can be customized similarly (check `mysnapshot` section in `src/io/snapshot_io.cpp`). (ToDo: create the template...)