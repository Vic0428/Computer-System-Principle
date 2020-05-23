# Fuchsia

## Overview

Google fuchsia is an open source capability-based operating system. 

## Getting started

1. Get the source code of `Fuchsia`

   - If you are in China, please try this [mirror](https://mirrors.sirung.org/fuchsia/source-code/) . 
   - Otherwise, directly visit [fuchsia official website](https://fuchsia.dev/fuchsia-src/). 

2. Setup environment variables (add to `~/.bashrc`)

   ```bash
   export PATH=$PATH:${fuchsia}/.jiri_root/bin
   source ${fuchsia}/scripts/fx-env.sh
   ```

3. For now, let’s first build the `zircon` kernel

   - Check what we can build?

     ```bash
     vic:~/fuchsia/zircon$ fx list-products
     bringup
     core
     router
     terminal
     workstation
     ```

     The "bringup" product identifies the Zircon components and excludes all other Fuchsia components from the build.

   - The defined board architecture are listed by running

     ```bash
     vic:~/fuchsia/zircon$ fx list-boards
     arm64
     as370
     c18
     chromebook-x64
     cleo
     *
     hikey960
     kirin970
     msm8998
     msm8x53-som
     mt8167s_ref
     qemu-arm64
     qemu-x64
     toulouse
     vim2
     vim3
     vs680
     x64
     x64-reduced-perf-variation
     ```

   - Define the target

     ```
     fx set bringup.x64
     ```

   - Now build!

     ```bash
     fx build
     ```

   - Now run on the `qemu`

     ```
     fx qemu
     ```

## Analysis of `fx`

1. Suppose the input command is `fx list-boards`, let’s look at `.jiri_root/fx`

   ```bash
   368 command_name="$1"
   369 command_path="$(find_executable ${command_name})"
   ```

   And the result shows that `command_path=./tools/devshell/list-boards`

2. So let’s take detail look at `fx set bringup.x64` and `fx build`

   It used `Gn` to build the project and two important directory
   `./out/default.zircon` and `./out/default`

3. Now let’s take a detail look at `fx qemu`

   It use following command to run the `qemu`

   ```bash
   "${FUCHSIA_DIR}/zircon/scripts/run-zircon" "${args[@]}" "$@"
   ```

   Let’s show the arguments of `run-zircon`

   ```bash
   -a x64 -q ${fuchsia}/prebuilt/third_party/qemu/linux-x64/bin -t ${fuchsia}/out/default/multiboot.bin --gic=3 -z ${fuchsia}/out/default/tmp.6OJ/fuchsia-ssh.zbi
   ```

   This following code can explain the goal of have`fuchsia-ssh.zbi`

   ```bash
   # Construction of a qcow image prevents qemu from writing back to the
   # build-produced image file, which could cause timestamp issues with that file.
   # Construction of the new ZBI adds //.ssh/authorized_keys for SSH access.
   imgdir="$(mktemp -d ${FUCHSIA_BUILD_DIR}/tmp.XXX)"
   if [[ ! -d "${imgdir}" ]]; then
     echo >&2 "Failed to create temporary directory"
     exit 1
   fi
   trap 'rm -rf "$imgdir"' EXIT
   
   kernelzbi="${imgdir}/fuchsia-ssh.zbi"
   args+=(-z "${kernelzbi}")
   ```

   Now look at `run-zircon` and learn the parameter meaning

   - `-a`: the architecture (`arm64`, or `x64`)
   - `-q`: location of `qemu`
   - `-t <binary>`: use this binary as the QEMU->ZBI trampoline
   - `-z <zbi>`: boot specified complete ZBI via trampoline
   - `--gic=<version>` use GIC v2 or v3

   Now show the full `qemu command`

   ```
   qemu-system-x86_64 
   -kernel /home/fwq/vic/csp/fuchsia/out/default/multiboot.bin 
   -initrd /home/fwq/vic/csp/fuchsia/out/default/tmp.H9V/fuchsia-ssh.zbi   
   -m 8192 
   -nographic 
   -nic none 
   -smp 4,threads=2 
   -machine q35 
   -device isa-debug-exit,iobase=0xf4,iosize=0x04 
   -cpu Haswell,+smap,-check,-fsgsbase 
   -append TERM=xterm-256color kernel.serial=legacy kernel.entropy-mixin=9b369f4dccf88dba1c5df1572d8aed2e4c75bd5a9e4871dd4797ae90895048f5 kernel.halt-on-panic=true 
   ```

4. Interesting part is `-kernel` and `-initrd`

   - First where is `fuchsia-ssh.zbi` from ?
     Look at the script `qemu`

     ```bash
     fx-zbi -o "${kernelzbi}" "${FUCHSIA_BUILD_DIR}/${IMAGE_ZIRCONA_ZBI}" \
       --entry "data/ssh/authorized_keys=${FUCHSIA_DIR}/.ssh/authorized_keys"
     ```

     And `${FUCHSIA_BUILD_DIR}/${IMAGE_ZIRCONA_ZBI} = ./out/default/bringup.zbi`

   - `multiboot.bin`

     ```
     GRUB -> multiboot.bin -> ./out/default/bringup.zbi
     ```

     [What’s multiboot?](https://www.gnu.org/software/grub/manual/multiboot/multiboot.html)

     

## Reference

1. [Build zircon](https://fuchsia.dev/fuchsia-src/development/kernel/getting_started)
2. [Github zircon-notes](https://github.com/PanQL/zircon-notes)

