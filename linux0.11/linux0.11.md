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

## Booting

1. Build a small `bootsect.s` (end with a infinite loop)

   ```assembly
   entry _start
   _start:
   ! Read cursor posiiton
       mov ah,#0x03
       xor bh,bh
       int 0x10
   
   ! Print the string in the screen
       mov cx,#24
       mov bx,#0x0007
       mov bp,#msg1
   ! es:bp : the address of string
       mov ax,#0x07c0
       mov es,ax
       mov ax,#0x1301
       int 0x10
   
   inf_loop:
       jmp inf_loop
   
   msg1:
   		.byte 13,10
   		.ascii "Loading system ..."
   		.byte 13,10,13,10
       
   .org 510
   boot_flag:
   		.word 0xAA55
   ```

   Next build the image (skip the first 32 bytes header)

   ```bash
   as86 -0 -a -o bootsect.o bootsect.s
   ld86 -0 -s -o bootsect bootsect.o
   dd bs=1 if=bootsect of=Image skip=32
   ```

   Now we can run `qemu` with this image

   

## Reference

1. [UTSC OS](http://staff.ustc.edu.cn/~ykli/os2020/)