The parameter files specify the input and output directory and format, and controls some behavior and performance of HBT+.
 
- Examples  

    You can find a few parameter files under config/, e.g., [config/Example.conf](https://github.com/Kambrian/HBTplus/blob/Hydro/configs/Example.conf).

- EAGLE runs (with apostle_io)

    To process EAGLE outputs, you will have to create a snapshotlist.txt file (due to the complex snapshot names) and place it under the subhalo path. There is a script toolbox/CreateSnapshotlist.py for creating the list.

- Yipeng's runs (with jing_io)

    A few extra parameters are needed for the IO. Check for example, config/8213.conf. Note that in order to use jing_io, you must set `USE_JING_IO=yes` in "Makefile.jing_io" when compiling; alternatively, you can compile with `make -e USE_JING_IO=yes`.