# Rifdock_install
## Install protocol for Rosetta 3.9 and Rifdock with basic explanation of computational concepts 

### 1. Install Rosetta 3.9 on linux

#### 1) Get liscence from RosettaCommons and download Rosetta 3.9
##### (1) Go to Rosetta commons homepage-Software-License and Download and apply for academic liscence. 
##### (2) After login with ID and password of the liscence, go to 'Downloads' and enter to Rosetta 3.9 - Download Rosetta 3.9. 
##### (3) Download Rosetta 3.9 source (2.8G) file (rosetta_src_3.9_bundle.tgz) and copy the source file to the folder on linux computer where the Rosetta 3.9 will be installed. 
 ##### (4) Install Rosetta by
     $ tar -xvzf rosetta_src_3.9_bundle.tgz
- The installation takes 10~20 min and rosetta_scr_2018.60072_bundle folder is generated.
 
#### 2) Compile Rosetta using Scons
##### (1) Move to 'source' folder by using command 
    $ cd rosetta_src_release_bundle/main/source
##### (2) Check 'Scons.py' in the source folder. This file is the software needed to compile Rosetta (already included in rosetta bundle in main/source folder). 
##### (3) Compile rosetta in MPI format(Massage Passing Interface) which is compatible to job-scheduler of CAE-simulator system.
    $ ./scons.py -j 10 mode=release bin/rosetta_scripts.mpi.linuxgccrelease extras=mpi
- '-j 10' means using 10 cores of CPU for compiling 
- 'mode=release' means compile with optimizations to produce faster version of rosetta
- 'mode=debug' or not menthioning any mode include additional checks which slows down Rosetta runs (not recommanded).
- 'extras=static' builds static binaries which could be useful to copy or run the apps on other systems.
- 'extras=mpi' compiles Rosetta in Massage Passing Interface format which support MPI run(Recommanded for CAE-Simulator system)
- If the extra option is mpi, use "bin/rosetta_scripts.mpi.linuxgccrelease", but if the option is static, then use "bin/rosetta_scripts.default.linuxgccrelease".
- There are additianl option to compile with specific cxx version. In that case, use the form like "$ ./scons.py -j 10 mode=release bin cxx=clang cxx_ver=4.5".
- This compiling process will took about 1h. The time could be reduced by increasing the number of CPU('-j 20' or more)
- It is general to compile Rosetta bundle with at least two version(static and mpi).
     
### 2. Install compatible GCC and Boost
- **To build RifDock, optain a copy of gcc with version >= 5.0**
- **Install Boost version 1.65 or later**
- Build a Rosetta cxx11_omp build with Ninja and cmake (Will be done in later stage)
#### 1) Install GCC >= 5.0
##### (1) Check the version of GCC which is compatible with Boost_1.65.1. The compatible GCC version is 5.4.0.
##### (2) Download GCC 5.4.0 from GNU server(https://ftp.gnu.org/gnu/gcc/gcc-5.4.0/). 
##### (3) Move the tar.gz file to rosetta folder and unzip the file with below command.
     $ tar -xvzf gcc-5.4.0.tar.gz
     
##### (4) Install GCC-5.4.0. (Ref: https://gcc.gnu.org/wiki/InstallingGCC)
     $ cd gcc-5.4.0
     $ ./contrib/download_prerequisites
     $ cd ..
     $ mkdir objdir
     $ cd objdir
     $../configure --prefix=$HOME/GCC-5.4.0 --enable-languages=c,c++,fortran,go
     $ make
     $ make install
 - I used "$../configure --prefix=$srgo/rosetta/GCC-5.4.0 --enable-languages=c,c++,fortran,go" because of the authority limitation.
 - In later stage, "CXX=/my/g++/version CC=/my/gcc/version ./ninja_build.py" will be used to build Rosetta cxx11_omp. For now, I'm not sure wheather it is really need to change entire gcc environment to alternative version.
##### (5) Change gcc version to the installed version
     $ 
  
#### 2) Install Boost 
##### (1) Download Boost version 1.65.1 from Boost C++ Library server(https://www.boost.org/users/history/version_1_65_1.html). 
##### (2) Move the downloaded file to the folder to install and unzip the file.
     $ cd rosetta/boost_1_65_1 
##### (3) Check the explanations and detailed options by
     $ ./bootstrap.sh --help
##### (4) Select the location to install the boost by using
     $ ./bootstrap.sh --prefix=srgo/rosetta/boost_build_1.65.1
##### (5) Start installation
     $ ./b2 install
- The installation process will took quite long time. 

### 3. Build a Rosetta cxx11_omp and rifdock
- To build RifDock, optain a copy of gcc with version >= 5.0
- Install Boost version 1.65 or later
- **Build a Rosetta cxx11_omp build with Ninja and cmake**
#### 1) Install Ninja and set the proper PATH
###### - In order to build rosetta cxx11_omp, Ninja shold be installed and proper PATH of Ninja need to be set.
##### (1) Download Ninja 1.10.2 from Git-hub(https://github.com/ninja-build/ninja/releases). (Ref:https://github.com/ninja-build/ninja/wiki, https://github.com/ninja-build/ninja) 
 
##### (2) Install Ninja
     $ tar xvzf- ninja-1.10.2.tar.gz
     $ cd ninja-1.10.2
     $ ./configure.py --bootstrap
     
##### (3) Set the proper PATH of Ninja
     $ echo $PATH
     $ export PATH=$PATH:/srgo/rosetta/ninja-1.10.2/
     $ echo $PATH
- export PATH=$PATH:new_adress_to_add
- Add PATH using 'export' is not permenant, so 
##### (4) Check the location of C++ and GCC (Ref: https://new.rosettacommons.org/docs/latest/build_documentation/Cxx11Support). 
     $ which -a c++
     $ which -a cc
     $ which -a gcc
- Those below are the returns of each commands.
- /usr/bin/C++
- /usr/bin/cc
- /usr/bin/gcc

##### (5) Enter to 'source' folder and build rosetta cxx11_omp using Ninja(Ref: See 'Extra tools' in https://ninja-build.org/manual.html)
     $ cd rosetta_src_release_bundle/main/source
     $ CXX=/usr/bin/c++ CC=/usr/bin/gcc ./ninja_build.py cxx11_omp -t rosetta_scripts -remake
- If the basic versio of C++ and gcc are compatible to build Rosetta cxx11_omp, use the basic address('/usr/bin/*')
- If there are specific requirement for alternative version of C++ or gcc, then install the compatible verison of compilers and change the address of the C++ and gcc to build rosetta cxx11_omp to alternative ones. - Not examined yet. 

##### (6) Copy rifdock repository 
     $ cd rifdock
     $ mkdir build
     $ cd build
     $ CXX=/usr/bin/c++ CC=/usr/bin/gcc CMAKE_ROSETTA_PATH=/Path/to/a/rosetta/main cmake -DCMAKE_BUILD_TYPE=Release
     $ make -j3 rif_dock_test rifgen
- If the rifdock is not link against cxx11_omp build, add "CMAKE_FINAL_ROSETTA_PATH=/Path/to/a/rosetta/main/source/cmake/build_my_custom_build_type" to the behind of the CMAKE_ROSETTA_PATH flag. 

##### (7) Unit Test
     $ make test_libscheme

### 4. Running Rifdock
##### The executable files for RifDock are built at:
$ rifdock/build/apps/rosetta/rifgen
$ rifdock/build/apps/rosetta/rif_dock/test

##### The unit tests executable file is at:
$ rifdock/build/schemelib/test/test_libscheme















#### + Basic programming concepts
compile: Change a file written in an language to another language\
build: a compiled version of a program \
bin: bin stands for binary file which is "non-text file". Binary file is compiled files which \
cxx: C++\
gcc: GNU Compiler Collection includes front ends(like UI for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages (libstdc++,...).\
front end: Front part of the system which is close to user. Starting point or input part of the system. GUI, FEP, etc.\
back end: The part that support the system on the backside like database. Only accessible to programmer or administor.\
MPI: Massage Passing Interface is a standardized and implantable message-passing standard which is designed to function of paralled computing architectures.
