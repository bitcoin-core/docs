Setting up RISC-V emulation for testing bitcoin
===============================================

This document describes some steps to set up qemu in user mode to emulate the
RISC-V architecture in user-space and pass through the system calls to the native
system.

This is efficient for testing because it makes it unnecessary to install a whole VM.

It also makes it possible to control extactly what libraries are available to prevent
external dependencies from leaking in.

These same steps work for other architectures but for sake of simplicity, I'll
only focus on RISC-V here. This comes up a lot for RISC-V specifically because
it's still so hard to find hardware for it.

Installing qemu
----------------

Many distributions don't provide packages for a qemu that supports RISC-V 
as upstream support for this has been merged somewhat recently. Here are some steps
to build one from source.

Dependencies for debian:

```bash
apt install libglib2.0-dev
```

Check out the repository and build, install:

```bash
git clone https://github.com/qemu/qemu.git 
cd qemu
./configure --prefix=/opt/qemu --target-list=riscv64-linux-user --disable-tools
make -j4
make install
```

(optional) add install path to `.profile` or such:
```bash
echo 'PATH="/opt/qemu/bin:$PATH"' >> ~/.profile
```

RISC-V dynamic linker
---------------------

qemu will look in `/usr/gnemul/qemu-riscv64/lib` by default for the dynamic
linker (ELF interpreter). This can be overridden with the environment variable
`QEMU_LD_PREFIX`.

    ld-2.29.9000.so -> ../lib64/ld-2.29.9000.so

You can copy these files from an existing RISC-V install. Alternatively, you
can download the dynamic linker and bare-bones shared libraries (copied from
Fedora rawhide) here:

```bash
mkdir -p /usr/gnemul
cd /usr/gnemul
wget https://dev.visucore.com/bitcoin/qemu-riscv64-libs.tar.gz
tar -zxvf qemu-riscv64-libs.tar.gz
```

(TODO: get this files directly from the packages instead of from wumpus' server)

RISC-V shared libraries
------------------------

As bitcoin doesn't distribute fully static executables you need part of the
sysroot of a RISC-V linux distribution. 

The dynamic linker will look in `/lib64/lp64d` by default for shared libraries
(not sure how to override this). 

These libraries are necessary:

    libm.so.6 -> libm-2.29.9000.so
    libpthread.so.0 -> libpthread-2.29.9000.so
    libatomic.so.1 -> libatomic.so.1.2.0
    libgcc_s.so.1 -> libgcc_s-9-20190503.so.1
    libc.so.6 -> libc-2.29.9000.so

To make them available you could download the archive of previous section and do:

```bash
ln -s /usr/gnemul/qemu-riscv64/lib64 /lib64/lp64d
```

Launching bitcoin
------------------

After this, bitcoin, or the unit tests, can be launched as normal with:

```bash
qemu-riscv64 bitcoind -datadir=...
```

(TODO: invoking the functional tests)

binfmt
------

note: It's possible to set up binfmt so that executing RISC-V ELF executables
automatically invokes them through qemu. How to do this differs per distribution.
I haven't set this up myself (I'm ok with prefixing `qemu-riscv64`).

