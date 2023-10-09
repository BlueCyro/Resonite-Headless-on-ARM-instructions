# How to build the libraries for ARM 64 bit

Hi! This is a short guide on how to build the native libraries required to run a Neos headless server on `aarch64` (Arm 64 bit) systems! Too long have we waited to fry eggs on our pis (or other assorted ARM chip ~~victims~~), so I took up the challenge of getting these libraries to compile and writing easy-to-follow instructions to do so. If you're omega lazy and don't want to try compiling these (though you may want to depending on what system you're on), you can try a couple pre-compiled variants I'll throw into the [releases](https://github.com/RileyGuy/Neos-Headless-on-ARM-instructions/releases) section of this repo.

## Prerequisites:
Some of these may have varying names depending on your distro's package manager, use your noggin to search for them

(Commonly `sudo apt search packagename` or `pacman -Ss packagename`)

Make sure the following are installed:
* clang
* cmake
* autoreconf
* make
* mono (To run the headless. As of writing this, version `6.12.0.182` works though slightly earlier or later versions may also function)


## Required libraries

*Tip: Whenever you see `make -jN` replace N with the number of cores the system you're compiling on has.*

__NOTE: When cloning, do `git clone --branch neos-release your://url.here` to get the proper branch!!__

I've hyperlinked the git repos in the table below, so you should just be able to right click -> copy them into the `your://url.here` portion of the command in the note above. I have specifically forked FreeImage and Crunch in order to fix them for ARM
| Library | Instructions |
| --- | --- |
| [FreeImage](https://github.com/Yellow-Dog-Man/FreeImage) | Edit `Makefile.gnu` and add `-DPNG_ARM_NEON_OPT=0` to the line that starts with `CFLAGS ?=`  as an option. Then run `make -jN` in the main directory, the library will end up in the 'Dist' folder of the main directory which you can navigate to by typing `cd Dist`. |
| [Crunch](https://github.com/RileyGuy/crunch) | Run `cmake -B build`, then cd into the 'build' directory and run `make -jN`. The library will end up directly in the build directory. |
| [Freetype](https://github.com/Neos-Metaverse/freetype) | Run `cmake -B build -D BUILD_SHARED_LIBS=true -D CMAKE_BUILD_TYPE=Release`, then cd into the 'build' directory and run `make -jN`. The library will end up directly in the build directory. |
| [Opus](https://github.com/Neos-Metaverse/opus) | If the `autogen.sh` file isn't executable already, mark is as such with `chmod +x autogen.sh` then run `./autogen.sh` and `./configure`. After that run `make -jN`. The library will be in a hidden directory called '.libs' so you can just type `cd .libs` to find it. |

## Optionals (though highly recommendables)
| Library | Instructions |
| --- | --- |
| [MSDFGen](https://github.com/Neos-Metaverse/msdfgen) | Run `cmake -B build`, then cd into the 'build' directory and run `make -jN`. The library will end up directly in the build directory. |
| [RNNoise](https://github.com/Neos-Metaverse/rnnoise) | Run `cmake -B build`, then cd into the 'build' directory and run `make -jN`. The library will end up directly in the build directory. |
| [Assimp](https://github.com/Neos-Metaverse/assimp) | (Pending successful compilation) |

Once you're done with a build, the library should be in the folder indicated for the instruction you followed. Typically these files will end in `.so`. For example: Opus builds a `libopus.so`, and FreeImage will build a `libfreeimage-x.xx.x.so` where `x` corresponds to the version numbers.

In the main root of the headless folder (the one containing `Neos.exe`) you should see similarly-named files also ending in `.so`. Once you've finished compiling, you're going to want to overwrite these with the versions you compiled. So for example you'd end up renaming `libfreeimage-x.xx.x.so` to `libFreeImage.so` and overwriting the one in the headless folder. Do this for all files.

Once these steps are complete, you can try firing up the headless by `cd`-ing into the directory and typing `mono Neos.exe` from the command line in the headless directory. If it runs, configure your headless as normal.

## Troubleshooting

### Compiler errors
May god have mercy on your soul. These can be pretty system-specific if you're building this on a pretty out-there processor or system (such as termux on android).

### DllNotFound errors
These errors are a bit easier to troubleshoot. Errors like this will likely appear in a threatening red text in your console if a library isn't being found by the headless. This just means that the server can't see a specific library and is usually a pretty easy fix (usually a misplaced/mis-named library). You can prepend `MONO_LOG_LEVEL=debug` and `MONO_LOG_MASK=dll` to the launch command such that it reads `MONO_LOG_LEVEL=debug MONO_LOG_MASK=dll mono Neos.exe` to get extended logging and more information about what exactly isn't loading.
