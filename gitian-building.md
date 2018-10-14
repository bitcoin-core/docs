Gitian building
================

*Setup instructions for a Gitian build of Bitcoin Core using a VM or physical system.*

Gitian is the deterministic build process that is used to build the Bitcoin
Core executables. It provides a way to be reasonably sure that the
executables are really built from the git source. It also makes sure that
the same, tested dependencies are used and statically built into the executable.

Multiple developers build the source code by following a specific descriptor
("recipe"), cryptographically sign the result, and upload the resulting signature.
These results are compared and only if they match, the build is accepted and provided
for download.

More independent Gitian builders are needed, which is why this guide exists.
It is preferred you follow these steps yourself instead of using someone else's
VM image to avoid 'contaminating' the build.

Table of Contents
------------------

- [Preparing the Gitian builder host](#preparing-the-gitian-builder-host)
- [Getting and building the inputs](#getting-and-building-the-inputs)
- [Building Bitcoin Core](#building-bitcoin-core)
- [Building an alternative repository](#building-an-alternative-repository)
- [Signing externally](#signing-externally)
- [Uploading signatures](#uploading-signatures)

Preparing the Gitian builder host
---------------------------------

The first step is to prepare the host environment that will be used to perform the Gitian builds.
This guide explains how to set up the environment, and how to start the builds.

Gitian builds are known to be working on recent versions of Debian, Ubuntu and Fedora.
If your machine is already running one of those operating systems, you can perform Gitian builds on the actual hardware.
Alternatively, you can install one of the supported operating systems in a virtual machine.

Any kind of virtualization can be used, for example:
- [VirtualBox](https://www.virtualbox.org/) (covered by this guide)
- [KVM](http://www.linux-kvm.org/page/Main_Page)
- [LXC](https://linuxcontainers.org/)

Please refer to the following documents to set up the operating systems and Gitian.

|                                   | Debian                                                                             | Fedora                                                                             |
|-----------------------------------|------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Setup virtual machine (optional)  | [Create Debian VirtualBox](./gitian-building/gitian-building-create-vm-debian.md) | [Create Fedora VirtualBox](./gitian-building/gitian-building-create-vm-fedora.md) |
| Setup Gitian                      | [Setup Gitian on Debian](./gitian-building/gitian-building-setup-gitian-debian.md) | [Setup Gitian on Fedora](./gitian-building/gitian-building-setup-gitian-fedora.md) |

Note that a version of `lxc-execute` higher or equal to 2.1.1 is required.
You can check the version with `lxc-execute --version`.
On Debian you might have to compile a suitable version of lxc or you can use Ubuntu 18.04 or higher instead of Debian as the host.

Non-Debian / Ubuntu, Manual and Offline Building
------------------------------------------------
The instructions below use the automated script [gitian-build.py](https://github.com/bitcoin/bitcoin/blob/master/contrib/gitian-build.py) which only works in Debian/Ubuntu. For manual steps and instructions for fully offline signing, see [this guide](./gitian-building/gitian-building-manual.md).

MacOS code signing
------------------
In order to sign builds for MacOS, you need to download the free SDK and extract a file. The steps are described [here](./gitian-building/gitian-building-mac-os-sdk.md). Alternatively, you can skip the OSX build by adding `--os=lw` below.

Initial Gitian Setup
--------------------
The `gitian-build.py` script will checkout different release tags, so it's best to copy it:

```bash
cp bitcoin/contrib/gitian-build.py .
```

You only need to do this once:

```
./gitian-build.py --setup satoshi 0.16.0rc1
```

Where `satoshi` is your Github name and `0.16.0rc1` is the most recent tag (without `v`). 

In order to sign gitian builds on your host machine, which has your PGP key, fork the gitian.sigs repository and clone it on your host machine:

```
git clone git@github.com:bitcoin-core/gitian.sigs.git
git remote add satoshi git@github.com:satoshi/gitian.sigs.git
```

Build binaries
-----------------------------
### What is code signing?
Before building the binaries, let's explain code signing.

Windows and OSX have code signed binaries, with private keys held by [Bitcoin Core Signing Association](https://bitcoincorecodesigning.org). Since you don't have those keys, you are not able to sign the binaries. 

This is done, because both Microsoft and Apple enforce signed binaries.

### Bitcoin Core process
The process is therefore as follows:

1. people deterministically build bitcoin without the codesigning and gitian-sign it
2. Bitcoin Core Signing Association codesigns the binaries, after there is a few gitian-signs
3. the codesignatures are then added to https://github.com/bitcoin-core/bitcoin-detached-sigs (note the different branches for each version)
4. other people can then codesign with those signatures, and then gitian-sign those binaries

Note that there is two separate, orthogonal concepts of "signing" - one is **gitian signing**, which is developers confirming, that the code on github produces the binary; and other is **codesigning**, which is signing the binary with Apple/Microsoft-provided key. 

You as a gitian signer can do the gitian signing - that is, sign the `.assert` files; if the binaries are already codesigned by Bitcoin Core Signing Association, you can attach their signatures; and then you can also check if the result of **that** produces the same binary and also sign the new `.assert`.

So the result of all this, from you, will be (for OS X and Windows) assert for non-codesigned binaries, then gpg signature for that, then assert for code-signed binaries and then gpg signature for that.

### Build binaries
To build the most recent tag, do:

 `./gitian-build.py --detach-sign --build --no-commit satoshi 0.16.0rc1`

To speed up the build, use `-j 5 -m 5000` as the first arguments, where `5` is the number of CPU's you allocated to the VM plus one, and 5000 is a little bit less than then the MB's of RAM you allocated.

If all went well, this produces a number of (uncommited) `.assert` files in the gitian.sigs repository.

### Sign asserts
Copy these uncommited changes to your host machine, where you can sign them:

```
#on host machine
NAME=satoshi
VERSION=0.17.0
for FILENAME in $( ls $VERSION-*/$NAME/*.assert )
do
gpg --output $FNAME.sig --detach-sign $FNAME 
done 
```

This will create the `.sig` files that can be committed together with the `.assert` files to assert your
Gitian build.

### Make a PR

Make a PR (both the `.assert` and `.assert.sig` files) to the
[bitcoin-core/gitian.sigs](https://github.com/bitcoin-core/gitian.sigs/) repository:

```
git checkout -b 0.16.0rc1-not-codesigned
git commit -S -a -m "Add $NAME 0.16.0rc non-code signed signatures"
git push --set-upstream $NAME 0.16.0rc1
```

You can also mail the files to Wladimir (laanwj@gmail.com) and he will commit them.

### Is the Bitcoin Core version code-signed already?
To know if there already is a detached signature, go to tree in bitcoin-detached-sigs repo - for example, https://github.com/bitcoin-core/bitcoin-detached-sigs/tree/v0.16.3 .

If the Bitcoin Core is on the website, it is most probably already codesigned.

### Make codesigned binaries, make PR

If there is a detached signature already, you can also make the codesigned binaries.

 `./gitian-build.py --detach-sign --sign --no-commit satoshi 0.16.0rc1`
 
Note that the `--sign` option does not actually sign anything; it attaches the detached signatures.

This will create another set of assert files (e.g. `signed` ones), and you should sign them too.

```
#on host machine
NAME=satoshi
VERSION=0.17.0
for FILENAME in $( ls $VERSION-*-signed/$NAME/*.assert )
do
gpg --output $FNAME.sig --detach-sign $FNAME 
done 
```

Make another pull request for these.
