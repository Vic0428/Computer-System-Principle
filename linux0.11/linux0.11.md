# Linux 0.11

## Overview

Learn the design and implementation of `Linux 0.11`

- The source code has been modified to be able to compiled by `> gcc-4.9`
- I have made a docker image which has essential package installed
  - Simply `docker pull vic0428/linux0.11`

## Startup

1. First get the source code

   ```bash
   wget https://git.lug.ustc.edu.cn/gloomy/ustc_os/raw/master/Linux-0.11-lab1.tar.gz
   ```

2. Pull the container

   ```
   docker pull vic0428/linux0.11
   ```

3. Run the container

   ```c++
   docker run -it --rm -v ${Linux-0.11}:/Linux-0.11 vic0428/linux0.11
   ```

   - In container, we can

     - `make` compile the image

     - `make debug` start gdb

       - We need to have a `.gdbinit`  in `Linux-0.11` source code directory

         ```bash
         set architecture i386:x86-64
         file tools/system
         target remote ${192.168.xxx.xxx}:1234
         ```

   - Outside container

     - `make start` => start the `qemu`

## Reference

1. [UTSC OS](http://staff.ustc.edu.cn/~ykli/os2020/)