This top-level README is not here on a permanent basis. It will eventually be replaced with the README in sources/application.

Pending organizational choices to be made with the team.

The original lpcxx-cmake README follows. It iwll be simplified and cut-down as constraints are imposed.
   ---J. Ian Lindsay


# lpc43xx-cmake
## Prerequisites
*Versions listed are verified as working.*
* CMake (v3.1.2): http://www.cmake.org/
* GNU ARM Toolchain:
 * LPCXpresso (v7.6.2): http://www.lpcware.com/lpcxpresso/download
* (Windows-only) GNU ARM Build Tools (v2.3): http://sourceforge.net/projects/gnuarmeclipse/files/Build%20Tools/
 * Ensure the path to the installed tools is added to the Windows `PATH` variable.
 * Alternatively, instead of installing the GNU ARM Build Tools, you can also simply add the path to the LPCXpresso-supplied tools to the Windows `PATH` variable: e.g. *C:\nxp\LPCXpresso_7.6.2_326\lpcxpresso\msys\bin*
* Python 3.4+ (v3.4.2) (if debugging within LPCXpresso is required): https://www.python.org/downloads/
 * Requires lxml package: http://lxml.de/

## Building
1) Clone this repository.  
2) Create a new folder next to the cloned repository (e.g. 'lpc43xx-cmake_build').  
3) Run CMake within the newly created folder to generate Makefiles and Eclipse project files (specify the correct path to the GNU ARM toolchain):  
```
$ cmake -DTOOLCHAIN_PREFIX="C:/nxp/LPCXpresso_7.6.2_326/lpcxpresso/tools" -G "Eclipse CDT4 - Unix Makefiles" -DCMAKE_ECLIPSE_VERSION=4.4.0 ../lpc43xx-cmake
```
4) Build project:  
```
$ make
```
5) **(OPTIONAL: If debugging within LPCXpresso is required)** Patch generated Eclipse .cproject file:  

Windows:
```
$ patch_cproject.bat
```
Linux/Mac:
```
$ patch_cproject.sh
```
**IMPORTANT:** Patch must be re-applied if CMake is re-run.

## CMake Arguments
A number of additional arguments can be specified to CMake to further configure the build:
* **TOOLCHAIN_PREFIX**: Path to the GNU ARM toolchain.
* **CMAKE_BUILD_TYPE**: Build type [Release, Debug]. *Default: Debug*
* **OUTPUT_NAME**: Set name of output files: e.g. ```OUTPUT_NAME```.axf, ```OUTPUT_NAME```.bin, ```OUTPUT_NAME```.hex, ```OUTPUT_NAME```.lst. *Default: ```CMAKE_PROJECT_NAME```*
* **CMAKE_ECLIPSE_MAKE_ARGUMENTS**: Arguments to pass to Eclipse make. Here the number of threads for parallel building can be specified. *Default: -j4*
* **CMAKE_ECLIPSE_VERSION**: Version of Eclipse to generate for. Specifying the correct version allows the CMake Eclipse Generator to use the latest Eclipse features. *LPCXpresso v7.6.2 uses Eclipse 4.4.0.*
* **CLIB**: C/C++ library to use [newlib, newlib-nano, redlib]. *Default: newlib-nano*
* **HOSTING**: Hosting settings for the build [none, nohost, semihosting]. *Default: nohost*
* **DEVICE**: Target device name [LPC4357]. *Default: LPC4357*
* **FLASHDRIVER**: LPCXpresso-supplied file which determines where the program will be flashed to during debugging. *Default: LPC18x7_43x7_2x512_BootA.cfx*
* **RESETSCRIPT**: LPCXpresso-supplied file used during debugging. *Default: LPC18LPC43InternalFLASHBootResetscript.scp*
* **PRINTF_FLOAT**: Enable/disable float format in printf (for newlib-nano and redlib only) [ON, OFF]. *Default: OFF*
* **SPRINTF_FLOAT**: Enable/disable float format in sprintf (for newlib-nano only) [ON, OFF]. *Default: OFF*
* **CHAR_PRINTF**: Enable/disable character-based printf (rather than string-based) (for redlib only) [ON, OFF]. *Default: OFF*
* **CPP**: Enable/disable C++ support [ON, OFF]. *Default: OFF*
* **CRP**: Enable/disable Code Read Protection [ON, OFF]. *Default: OFF*
* **BSP_NAME**: Board Support Package to use. *This argument is optional: just ensure that the path to the BSP matches sources/bsp/```BSP_NAME```/```BSP_VERSION```. Note: If both ```BSP_NAME``` and ```BSP_VERSION``` are not specified, no BSP will be used.*
* **BSP_VERSION**: Board Support Package version to use. *This argument is optional: just ensure that the path to the BSP matches sources/bsp/```BSP_NAME```/```BSP_VERSION```. Note: If both ```BSP_NAME``` and ```BSP_VERSION``` are not specified, no BSP will be used.*
* **LPCOPEN_VERSION**: LPCOpen version to use in the BSP (or in the application if no BSP is present). The value must match the name of a subdirectory in the *sources/lpcopen* folder. *Default: 2.12*
* **LINKER_SCRIPT_DIR**: Specify which linker scripts to use. This allows custom linker scripts to be used. IMPORTANT: Ensure that the directory layout and filenames within ```LINKER_SCRIPT_DIR``` matches that of *platform/lpc43xx/ldscripts* - a good starting point is to copy this directory and then make your required changes to the linker script files within. *If this argument is not specified, the default linker scripts contained in platform/lpc43xx/default/ldscripts are used.*

### OpenOCD
* **OPENOCD_BINARY**: Specify the path to the OpenOCD binary executable. If specified and valid, target device can be flashed via ```make flash```.
* **OPENOCD_CONFIG**: Specify the OpenOCD configuration file to use. This value must match the name of a file in the *debug* directory. *Default: stlink-v2_lpc43xx.cfg*
* **OPENOCD_TRANSPORT**: Specify the transport to use with the OpenOCD configuration. Ensure the configuration supports the transport you specify. *Default: hla_jtag*

## Make Targets
* ```make```: Builds the entire project and outputs an .axf (ARM Executable Format) file - this is actually a ELF/DWARF image. Equivalent to ```make all```.
* ```make <target_name>```: Build only a specific target (e.g: ```make lpc_chip_43xx```)
* ```make clean```: Cleans all target output files
* ```make hex```: Generate a hex file
* ```make bin```: Generate a binary file which includes the required checksum (See: http://www.lpcware.com/content/faq/lpcxpresso/image-checksums)
* ```make lst```: Generate a listings file
* ```make flash```: Flash the binary file to the target via OpenOCD. Only available if the CMake argument ```OPENOCD_BINARY``` is specified and valid. Note: This target calls ```make bin``` before flashing.
* ```make erase```: Erases the target device flash via OpenOCD. Only available if the CMake argument ```OPENOCD_BINARY``` is specified and valid. Note: The flash bank specified by ```FLASHDRIVER``` will be erased only.

## Linker Script
### Inputs
The linker script is generated dynamically and changes depending on the following CMake arguments:

* ```CLIB```
* ```HOSTING```
* ```CPP```
* ```CRP```

### Format
The generated linker script uses the ```INCLUDE``` command to add all correct components in order to create a complete linker script:

```
INCLUDE "path_to_LIBRARY_file.ld"
INCLUDE "path_to_CPP_LIBRARY_file.ld" /* Only added if C++ is enabled */
INCLUDE "path_to_MEMORY_file.ld"
INCLUDE "path_to_SECTIONS_file.ld"
```
#### Library File
* Tells the linker which C library to link.
* **DEPENDS:** ```CLIB```, ```HOSTING```

#### CPP Library File
* Tells the linker to link the C++ library.
* Tells the linker to link the required initialization and termination routines for C++ constructors and destructors respectively.
* **DEPENDS:** ```CPP```

#### Memory File
* Defines each memory region (base address and size).
* Defines symbols for the top of each memory region.
* **DEPENDS:** None - this file is boilerplate (i.e. it does not depend on any CMake arguments).

#### Sections File
* Tells the linker where to put each section (e.g. text, rodata, bss, data) in memory.
* **DEPENDS:** ```CPP```, ```CRP```
    * If ```CPP``` is enabled, where to put the required C++ library initialization code is included.
    * If ```CRP``` is enabled, where to put the required CRP value in flash is included.
