# Rifdock_install
## Install protocol for Rosetta 3.9 and Rifdock with basic explanation of computational concepts 

### 1. Install Rosetta 3.9 on linux

#### 1) Get liscence from RosettaCommons and download Rosetta 3.9
 (1) Go to Rosetta commons homepage-Software-License and Download and apply for academic liscence. \
 (2) After login with ID and password of the liscence, go to 'Downloads' and enter to Rosetta 3.9 - Download Rosetta 3.9. \
 (3) Download Rosetta 3.9 source(2.8G) file(rosetta_src_3.9_bundle.tgz) and copy the source file to the folder on linux computer where the Rosetta 3.9 will be installed. \
 ##### (4) Install Rosetta by
     $ tar -xvzf rosetta_src_3.9_bundle.tgz
- The installation takes 10~20 min and rosetta_scr_2018.60072_bundle folder is generated.
 
#### 2) Compile Rosetta using Scons
##### (1) Move to 'source' folder by using command 
    $ cd rosetta_src_release_bundle/main/source
 (2) Check 'Scons.py' in the source folder. This file is the software needed to compile Rosetta (already included in rosetta bundle in main/source folder). \
 ##### (3) Compile rosetta in MPI format(Massage Passing Interface) which is compatible to jobscheduler of CAE-simulator system.
    $ ./scons.py -j 10 mode=release bin/rosetta_scripts.mpi.linuxgccrelease extras=mpi
- '-j 10' means using 10 cores of CPU for compiling \
- 'mode=release' means compile with optimizations to produce faster version of rosetta \
- 'mode=debug' or not menthioning any mode include additional checks which slows down Rosetta runs (not recommanded).\
- This compiling process will took about 1h. The time could be reduced by increasing the number of CPU('-j 20' or more)
     
### 2. Install compatible GCC and Boost

#### 1) Install GCC >= 5.0
 (1) Check the version of GCC which is compatible with Boost_1.65.1. The compatible GCC version is 5.4.0.\
 (2) Download GCC 5.4.0 from GNU server(https://ftp.gnu.org/gnu/gcc/gcc-5.4.0/). \
 (3) Move the tar.gz file to rosetta folder and unzip the file with below command.
     $ tar -xvzf gcc-5.4.0.tar.gz
##### (4) Install GCC-5.4.0. (Ref: https://gcc.gnu.org/wiki/InstallingGCC)
     $ cd gcc-5.4.0
     $ ./contrib/download_prerequisites
     $ cd ..
     $ mkdir objdir
     $ cd objdir
     $../configure --prefix=$srgo/GCC-5.4.0 --enable-languages=c,c++,fortran,go
     $ make
     $ make install
  (5)Change gcc version to the installed version(?)
  
#### 2) Install Boost 
 (1) Download Boost version 1.65.1 from Boost C++ Library server(https://www.boost.org/users/history/version_1_65_1.html). \
#### (2) Move the downloaded file to the folder to install and unzip the file.
     $ cd rosetta/boost_1_65_1 
#### (3) Check the explanations and detailed options by
     $ ./bootstrap.sh --help
#### (4) Select the location to install the boost by using
     $ ./bootstrap.sh --prefix=srgo/rosetta/boost_build_1.65.1
#### (5) Start installation
     $ ./b2 install
- The installation process will took quite long time. 

### 3. Build a Rosetta cxx11_omp and rifdock
#### 1) Install Ninja and set the proper PATH
- In order to build rosetta cxx11_omp, Ninja shold be installed and proper PATH of Ninja need to be set.
 (1) Download Ninja 1.10.2 from Git-hub(https://github.com/ninja-build/ninja/releases). (Ref:https://github.com/ninja-build/ninja/wiki, https://github.com/ninja-build/ninja) \
 
##### (2) Install Ninja
     $ tar xvzf- ninja-1.10.2.tar.gz
     $ cd ninja-1.10.2
     $ ./configure.py --bootstrap
     
##### (3) Set the proper PATH of Ninja
     $ cp ninja /usr/bin/
     
##### (4) Check the location of C++ and GCC. \
     $ which -a c++
     $ which -a cc
     $ which -a gcc
- /usr/bin/C++
- /usr/bin/cc
- /usr/bin/gcc

##### (5) Enter to 'source' folder and build rosetta cxx11_omp using Ninja
     $ cd rosetta_src_release_bundle/main/source
     $ CXX=/usr/bin/c++ CC=/usr/bin/gcc ./ninja_build.py cxx11_omp -t rosetta_scripts -remake

##### (6) Copy rifdock repository 
     $ cd rifdock
     $ mkdir build
     $ cd build
     $ CXX=/usr/bin/c++ CC=/usr/bin/gcc CMAKE_ROSETTA_PATH=/Path/to/a/rosetta/main cmake -DCMAKE_BUILD_TYPE=Release
     $ make -j3 rif_dock_test rifgen
- If the rifdock is not link against cxx11_omp build, add "CMAKE_FINAL_ROSETTA_PATH=/Path/to/a/rosetta/main/source/cmake/build_my_custom_build_type" behind CMAKE_ROSETTA_PATH flag. 

##### (7) Unit Test
     $ make test_libscheme

### 4. Running Rifdock
##### The executable files for RifDock are built at:
- $ rifdock/build/apps/rosetta/rifgen
- $ rifdock/build/apps/rosetta/rif_dock/test

##### The unit tests executable file is at:
- $ rifdock/build/schemelib/test/test_libscheme















#### + Basic programming concepts
compile: Change a file written in an language to another lanhuage\
bin: bin stands for binary file which is "non-text file". Binary file is compiled files which \
cxx: C++\
gcc: GNU Compiler Collection includes front ends(like UI for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages (libstdc++,...).\
front end: Front part of the system which is close to user. Starting point or input part of the system. GUI, FEP, etc.\
back end: The part that support the system on the backside like database. Only accessible to programmer or administor.\
MPI: Massage Passing Interface is a standardized and implantable message-passing standard which is designed to function of paralled computing architectures.
