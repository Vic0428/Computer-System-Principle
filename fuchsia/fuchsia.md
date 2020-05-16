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

     



## Reference

1. [Build zircon](https://fuchsia.dev/fuchsia-src/development/kernel/getting_started)
2. 