# RifDock Setup Protocol
- Setup protocol to build Rosetta and RifDock
- Tested on Red Hat 8.3.1 CentOS 8 
- Written by SeongRyeong Go on 28 Apr 2022.
- **If you want to directly see the method to build RifDock, see #7(https://url.kr/e8jszh)**

### These are the required files to design protein binders with RifDock
- Rosetta_3.13 (Recent Rosetta later than 2020, HDF5 is needed)
- DAIphaBall(Part of Rosetta)
- PyRosetta4
- PatchDock
- PsiPred
- silent_tools
- motif_clustering/cluster
- Rosetta_3.9 compiled as cxx11_omp (Re2c and Ninja are needed)
- RifDock (Boost 1.65.0 is needed)

### Notice
- In this protocol, the command line which starts with **"#"** means **"root"** or **"administer"**, and the command line which starts with **"$"** means **"local user"**.

### Directory Structure
        
        bin/    home/   root/    usr/  

        /root/packages/
        hdf5-1.12.1/     boost-1.65.0/    

        /home/users/local_user/packages/
        ninja-1.10.2/    re2c-1.0.3/

        /home/users/local_user/rosetta/
        rosetta_src_release_3.9/     PyRosetta4/      rosetta_src_release_3.13/    rifdock/     
        psipred/     PatchDock/       ppi_tools/       silent_tools/      ncbi-blast/    ncbi_tools/

        /usr/local/
        bin/   lib/  include/   hdf5/

- These are the major directories which are involved in setting up rifdock

### 1. Install HDF5 and Rosetta 3.13
#### 1) Install HDF5 (Ref: https://fossies.org/linux/hdf5/release_docs/INSTALL)
##### (1) Download HDF5 source file from The HDF5 Group server(https://www.hdfgroup.org/downloads/hdf5) . - Install ver 1.12.1.

##### (2) Login as administer and make the "packages" directory (This directory will be used to install new libraries, compilers, and etc).
    # mkdir packages
    # cd packages          // pwd is /root/packages/
     
##### (3) Move the downloaded file to the directory to install the file and unpack the tar file.             
    # tar -xvzf hdf5-1.12.1.tar.gz

##### (3) Compile HDF5.
    # cd hdf5-1.12.1
    # ./configure --prefix=/usr/local/hdf5

- If you compile HDF5 without any option("--enable-cxx" or "--enable-fortran"), it will use a basic compiler in your environment (like GCC) and most of the other options will be set as "disable". 
- If you want to see more options use "./configure --help".
- You can specify the location to install HDF5 bin, include, lib by using "--prefix=/path/to/install/hdf5/" flag. 

##### (4) Build and install HDF5
    # make
    # make install
    
- There could be some warning messages during the make step, but it will be okay if no "ERROR:~" or "fatal error" messages came out.

##### (5) Check whether the installation was successful.
    # cd /usr/local/hdf5
- Check wether "bin", "include", "lib" directory and the files are generated under /usr/local/hdf5.

##### (6) Login as local user and set the PATH and LD_LIBRARY_PATH in .bashrc
    $ cd ~
    $ vi .bashrc
- Add the address of hdf5/bin to PATH, and add hdf5/include and hdf5/lib to LD_LIBRARY_PATH
    
            # .bashrc

            # Source global definitions
            if [ -f /etc/bashrc ]; then
                . /etc/bashrc
            fi

            # User specific environment
            PATH="$HOME/.local/bin:$HOME/bin:$PATH"
            export PATH
            PATH=$PATH:/usr/local/hdf5/bin
            export PATH

            LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/hdf5/lib:/usr/local/hdf5/include
            export LD_LIBRARY_PATH


            # Uncomment the following line if you don't like systemctl's auto-paging feature:
            # export SYSTEMD_PAGER=

            # User specific aliases and functions

##### (7) Check wether $PATH and $LD_LIBRARY_PATH are updated
    $ echo $PATH
    $ echo $LD_LIBRARY_PATH
- If PATH is updated correctly, you can proceed to rosetta install and compilation.
- Do not add multiple versions of hdf5 to PATH or LD_LIBRARY_PATH. It will make a fatal error.
- If you want to install HDF5 more simply, then you can download "the pre-compiled HDF5 library" and use it without the compiling process. In this case, unpack the file to your local user directory and set the correct PATH to 'hdf5/bin' and LD_LIBRARY_PATH to 'hdf5/include' and 'hdf5/lib'.
- Because the addresses are added to ".bashrc", it will be maintained after you close your terminal or open a new terminal. If you don't need hdf5 anymore, remove the address to hdf5 binary and library from ".bashrc".

#### 2) Install Rosetta 3.13 and compile with HDF5
- Check whether the version of compilers in your machine and environment supports Rosetta before the installation (https://new.rosettacommons.org/docs/latest/build_documentation/Cxx11Support).
- GCC/g++: Version 4.8 or later (https://gcc.gnu.org/releases.html)
- Clang/llvm on Linux: Version 3.3 or later (https://releases.llvm.org/download.html)

##### (1) Go to Rosetta commons and apply for academic liscence (https://www.rosettacommons.org/software/license-and-download). 

##### (2) After login in with the ID and password of the license, go to 'Downloads' and enter Rosetta 3.13 - Download Rosetta 3.13. 

##### (3) Download Rosetta 3.13 source (5.2G) file (rosetta_src_3.13_bundle.tgz) and copy the source file to the folder on the Linux computer where the Rosetta 3.13 will be installed (https://www.rosettacommons.org/downloads/academic/3.13/). 

##### (4) Unpack the tar file.
     $ tar -xvzf rosetta_src_3.13_bundle.tgz
- The installation takes 20~30 min and rosetta_scr_release_bundle folder will be generated.

##### (5) Move to 'source' folder.
    $ cd rosetta_src_release_bundle/main/source

##### (6) Check 'Scons.py' is in the source folder. This file is the software needed to compile Rosetta (already included in the rosetta bundle in main/source folder). 

##### (7) Compile rosetta in HDF5 format which is the basic form of Rosetta.
    $ ./scons.py -j10 mode=release extras=hdf5 rosetta_scripts

- '-j 10' means using 10 cores of CPU for compiling 
- 'mode=release' means compile with optimizations to produce a faster version of rosetta
- 'mode=debug' or not mentioning any mode include additional checks which slow down Rosetta runs (not recommended).
- 'extra=hdf5' option is used to build basic rosetta. If you use this option, add 'rosetta_scripts' at the end of the line.
- 'extras=static' builds static binaries which could be useful to copy or run the apps on other systems.
- 'extras=mpi' compiles Rosetta in Massage Passing Interface format which supports MPI run (Recommended for CAE-Simulator system)
- If the extra option is mpi, use "bin/rosetta_scripts.mpi.linuxgccrelease", but if the option is static, then use "bin/rosetta_scripts.default.linuxgccrelease".
- There are additional option to compile with specific cxx version. In that case, use the form like "$ ./scons.py -j 10 mode=release bin cxx=clang cxx_ver=4.5".
- This compiling process will take about 1h. The time could be reduced by increasing the number of CPUs ('-j 20' or more)
- It is general to compile Rosetta bundle with at least two versions (static and mpi).

##### (8) Check the compile result. If the compile was successfully done, the last sentence of the terminal shows the result like below.
    "Install file: "build/src/release/linux/4.18/64/x86/gcc/8/default/rosetta_scripts.default.linuxgccrelease" as "bin/rosetta_scripts.default.linuxgccrelease"
    scons: done building targets.
    
##### (9) If the compile was successful, there will be "rosetta_scripts.default.linuxgccrelease" and "rosetta_scripts.linuxgccrelease" in /main/source/bin.

### 2. Install DAIphaBall
##### (1) DAIphaBall is a part of Rosetta. Move to rosetta_src_release/main/source/external/DAIphaBall.

##### (2) Type "make" to build DAIphaBall.
    $ make

### 3. Install PyRosetta
##### (1) Get the license for PyRosetta in Rosetta Commons(https://www.rosettacommons.org/software/license-and-download)

##### (2) Check the Python version installed on the Linux computer.
    $ python --version
    
##### (3) Download proper version of PyRosetta4 in PyRosetta server(https://graylab.jhu.edu/download/PyRosetta4/archive/release/)

- Release : Speed optimized build for production runs.
- MinSizeRel : Build optimized to reduce memory which could be suitable for low memory systems.
- Debug : Binaries compiled in debug mode with additional asserts enabled and with debug-info compiled in. Build for debugging
- Python-x.x versions : The builds of PyRosetta for the specific Python version.
- "PyRosetta4.Release.python36.linux.release-316.tar.bz2" is selected for installation.

##### (4) Move the downloaded file to the directory to install PyRosetta and unzip the file.
    $ tar -vjxf PyRosetta4.Release.python36.linux.release-316.tar.bz2
    
##### (5) Enter to PyRosetta directory and type
    $ cd setup
    $ python setup.py install
    
- **This step requires administrative access. You need to sign in as administer or "root" account.**
##### (6) Check whether the PyRosetta is successfully installed. If it was successful, you can see these sentences in the last sentence of the installation.
    Installed /usr/local/lib/python3.6/site-package/pyrosetta-2022.14+release.d95c942-py3.6-linux-x86_64.egg
    Processing dependencies for pyrosetta==2022.14+release.d95c942
    Finished processing dependencies for pyrosetta==2022.14+release.d95c942

### 4. Install PatchDock
###### (1) Download PatchDock from PatchDock server(https://bioinfo3d.cs.tau.ac.il/PatchDock/).

###### (2) Unzip the file by
    $ unzip patch_dock_download.zip

###### (3) PatchDock do not require further install process. Read 'README.md' to run PatchDock.

### 5. Install PsiPred
#### 1) Install PsiPred
##### (1) Download PsiPred source code from Git-hub pirspred/pirspred (https://github.com/psipred/psipred).

##### (2) Move the zip file to the directory to install psipred in the Linux computer.

##### (3) Unpack the zip file by
    $ unzip psipred.zip
    
##### (4) Compile the software by
    $ cd psipred
    $ cd src
    $ make
    $ make install
    $ export PATH=$PATH:/path/to/psipred

##### (5) Test whether the psipred is run
    $ cd example
    $ runpsipred example.fasta
- If you didn't install PSI-BLAST, there will be the error that cannot find .blast files to run the psipred.

#### 2) Install PSI-BLAST and Impala from the NCBI toolkit
##### (1) Download Standalone PSI-BLAST from NCBI FTP server (https://ftp.ncbi.nih.gov/blast/executables/blast+/LATEST/).
##### (2) Unpack ncbi-blast.tar.gz file.
    $ cd rosetta
    $ tar -xvzf ncbi-blast-2.13.0+-x64-linux.tar.gz
    $ mv ncbi-blast-2.13.0+ blast                    // naming change from "ncbi~" to "blast"
    
##### (3) Check wether psiblast is in the bin directory.
    $ cd blast
    $ cd bin
    blastdb_aliastool
    blastdbcheck
    blastcmd
    ...
    psiblast
    ...
    update_blastdb.pl
    windowmasker
   
#### 3) Install Sequence Data Bank (UNIREF90)
##### (1) Download UNIREF90 from Uniprot DB server https://ftp.uniprot.org/pub/databases/uniprot/uniref/uniref90/).
- The size of uniref90.fasta.gz is 32.0G and that of uniref90.xml.gz is 49.1G. So, it will take a few hours to download the DB files.

##### (2) make uniref90 directory and unpack gz files.
    $ mkdir uniref90
    $ cd uniref90
    $ gzip -d uniref90.fasta.gz
    $ gzip -d uniref90.xml.gz
    
#### 4) Run psipred
    $ runpsipred example.fasta

    Running PSI-BLAST with sequence example.fasta ...
    Predicting secondary structure...
    Pass1 ...
    Pass2 ...
    Cleaning up ...
    Final output file: example.horiz
    Finished.

    That's it - you can then look at the output:

    $ more example.horiz




### 6. Install Boost and Ninja to build RifDock
- To build RifDock, obtain a copy of gcc with version >= 5.0
- Install Boost version 1.65 or later
- Tested CentOS8(Red Hat gcc 8.3.1) with Boost-1.65.0

#### 1) Install Boost
**Notification: I recommend you to log in as "root" or "administer" to the properly install boost.**

##### (1) Download Boost version 1.65.0 from Boost C++ Library server(https://www.boost.org/users/history/version_1_65_0.html). 
- Latest version of Boost (1.74.0 or 1.78.0) was not compatible with Rifdock.

##### (2) Move the downloaded file to the folder to install and unpack the file (Ref: https://valelab4.ucsf.edu/svn/3rdpartypublic/boost/more/getting_started/windows.html).
     # cd /root/
     # mkdir packages
     # cd packages
     # tar -xvzf boost_1_65_0.tar.gz
     # cd boost_1_65_0 

##### (3) Select the location to install the boost by using
     # ./bootstrap.sh --prefix=/path/to/install/boost/
- **If you are logged in as "root" or administer and want to make them automatically added to /usr/local directory, you don't need to add --prefix option. Check whether libboost_xxx.so and libboost_xxx.so.1.65.0 files are generated in /usr/local/lib and /usr/local/include.**

##### (4) Start installation
     # ./b2 install
- The installation process will take a quite long time. 
- Even though there could be some minor errors like "warning: unnecessary parentheses in the declaration of 'assert_mot_arg' [-Wparentheses]", it didn't make any error during compiling Rifdock.
- **Caution: If multiple versions of boost are installed, it makes an error when compiling RifDock. So, if you want to use another version of boost, it is recommended to delete the previous boost (check /usr/local/include /usr/local/lib /usr/lib) and install a new one.**

#### 2) Install re2c - Needed to run Ninja
##### (1) Download re2c from downloading site (https://opensuse.pkgs.org/15.3/opensuse-oss-x86_64/re2c-1.0.3-1.18.aarch64.rpm.html).
##### (2) Install re2c by using following commands (Ref: https://re2c.org/build/build.html).
    $ tar -xvzf re2c-1.0.3.tar.gz
    $ cd re2c-1.0.3
    $ mkdir build
    $ cd build
    $ ../configure && make && make install
##### (3) Check whether the installation was successful.
    $ make test
    
#### 3) Install Ninja
###### - In order to build rosetta cxx11_omp, Ninja shold be installed and proper PATH of Ninja need to be set.
##### (1) Download Ninja 1.10.2 from Git-hub(https://github.com/ninja-build/ninja/releases). (Ref:https://github.com/ninja-build/ninja/wiki, https://github.com/ninja-build/ninja) 
 
##### (2) Install Ninja
     $ tar xvzf- ninja-1.10.2.tar.gz
     $ cd ninja-1.10.2
     $ ./configure.py --bootstrap
     
##### (3) Set the proper PATH of Ninja (Necessary)
     $ echo $PATH
     $ export PATH=$PATH:/home/users/local/ninja-1.10.2/
     $ echo $PATH
- You can add PATH by using "export PATH=$PATH:/new_adress_to_add"

### 7. Build Rosetta cxx11_omp and RifDock
- This is the final stage of installing RifDock
#### 1) Build Rosetta 3.9 as cxx11_omp using Ninja
##### (1) Download Rosetta 3.9 source (2.8G) file (rosetta_src_3.9_bundle.tgz) and copy the source file to the directory where the Rosetta 3.9 will be installed. 

##### (2) Unpack Rosetta
     $ tar -xvzf rosetta_src_3.9_bundle.tgz

##### (3) Move to main/source folder
     $ cd rosetta_src_release_bundle/main/source
     
##### (4) build rosetta cxx11_omp using Ninja(Ref: See 'Extra tools' in https://ninja-build.org/manual.html)
     $ cd rosetta_src_release_bundle/main/source
     $ CXX=/usr/bin/g++ CC=/usr/bin/gcc ./ninja_build.py cxx11_omp -t rosetta_scripts -remake
- You can check the location of C++ and GCC by
     $ which -a c++
     $ which -a cc
     $ which -a gcc
- It is not necessary to compile rosetta 3.9 with HDF5 or others while building RifDock.

#### 2) Build RifDock
##### (1) Copy rifdock repository and build Rifdock (https://github.com/rifdock/rifdock)
     $ unzip rifdock-master.zip
     $ mv rifdock-master rifdock
     $ cd rifdock
     $ mkdir build
     $ cd build
     $ CXX=/usr/bin/g++ CC=/usr/bin/gcc CMAKE_ROSETTA_PATH=/Path/to/a/rosetta/main CMAKE_FINAL_ROSETTA_PATH=/Path/to/a/rosetta/main/source/cmake/build_cxx11_omp cmake .. -DCMAKE_BUILD_TYPE=Release
     $ make -j10 rif_dock_test rifgen
- When later version of Boost(Boost-1.74.0 or Boost-1.78.0) was used insted of Boost-1.65.0, it made BOOST_BITMASK error during "make" step.

##### [If you build the RifDock correctly, you can see the building process like this.]
    [srgo@anode0 rifdock]$ mkdir build
    [srgo@anode0 rifdock]$ cd build/
    [srgo@anode0 build]$ CXX=/usr/bin/g++ CC=/usr/bin/gcc CMAKE_ROSETTA_PATH=/home/users/srgo/rosetta/rosetta_src_2018.09.60072_bundle/main CMAKE_FINAL_ROSETTA_PATH=/home/users/srgo/rosetta/rosetta_src_2018.09.60072_bundle/main/source/cmake/build_cxx11_omp cmake .. -DCMAKE_BUILD_TYPE=Release
    -- The C compiler identification is GNU 8.3.1
    -- The CXX compiler identification is GNU 8.3.1
    -- Check for working C compiler: /usr/bin/gcc
    -- Check for working C compiler: /usr/bin/gcc -- works
    -- Detecting C compiler ABI info
    -- Detecting C compiler ABI info - done
    -- Detecting C compile features
    -- Detecting C compile features - done
    -- Check for working CXX compiler: /usr/bin/g++
    -- Check for working CXX compiler: /usr/bin/g++ -- works
    -- Detecting CXX compiler ABI info
    -- Detecting CXX compiler ABI info - done
    -- Detecting CXX compile features
    -- Detecting CXX compile features - done
    build type .................... Release
    python version ................ 
    c++ standard .................. c++11
    path to rosetta binaries ......     /home/users/srgo/rosetta/rosetta_src_2018.09.60072_bundle/main/source/cmake/build_cxx11_omp
    install prefix ................ /usr/local
    -- Found PythonInterp: /usr/bin/python (found version "3.6.8") 
    -- Looking for pthread.h
    -- Looking for pthread.h - found
    -- Looking for pthread_create
    -- Looking for pthread_create - not found
    -- Looking for pthread_create in pthreads
    -- Looking for pthread_create in pthreads - not found
    -- Looking for pthread_create in pthread
    -- Looking for pthread_create in pthread - found
    -- Found Threads: TRUE  
    -- Performing Test HAS_LTO_FLAG
    -- Performing Test HAS_LTO_FLAG - Success
    using python include .......... PYTHON_INCLUDE_DIR-NOTFOUND/python3.5m
    using python lib dir .......... 
    using python lib .............. -lpython3.5m
    install python libs to ........ /usr/local/lib/python
    process pysetta source file: _pysetta_core_import_pose.cc
    process pysetta source file: _pysetta_core_pose.cc
    process pysetta source file: _pysetta_devel.cc
    riflib exe: test_librosetta
    riflib exe: rifgen
    riflib exe: rif_dock_test
    riflib exe: scheme_make_bounding_grids
    -- Configuring done
    -- Generating done
    -- Build files have been written to: /home/users/srgo/rosetta/rifdock/build
    [srgo@anode0 build]$ make -j10 rif_dock_test rifgen
    Scanning dependencies of target riflib
    [  3%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/BurialManager.cc.o
    [  6%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/DonorAcceptorCache.cc.o
    [  9%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/EtableParams_init.cc.o
    [  9%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/HBondedPairGenerator.cc.o
    [ 12%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/HydrophobicManager.cc.o
    [ 15%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/RifFactory.cc.o
    [ 18%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/RotamerGenerator.cc.o
    [ 18%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/UnsatManager.cc.o
    [ 21%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rosetta_field.cc.o
    [ 24%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rotamer_energy_tables.cc.o
    [ 27%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/seeding_util.cc.o
    [ 27%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/util.cc.o
    [ 30%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/util_complex.cc.o
    [ 33%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rif/RifGeneratorApoHSearch.cc.o
    [ 33%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rif/RifGeneratorSimpleHbonds.cc.o
    [ 36%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rif/RifGeneratorUserHotspots.cc.o
    [ 39%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rif/make_hbond_geometries.cc.o
    [ 42%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rif/requirements_util.cc.o
    [ 42%] Building CXX object  apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/CompileAndFilterResultsTasks.cc.o
    [ 45%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/HSearchTasks.cc.o
    [ 48%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/HackPackTasks.cc.o
    [ 51%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/MorphTasks.cc.o
    [ 51%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/OutputResultsTasks.cc.o
    [ 54%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/RosettaScoreAndMinTasks.cc.o
    [ 57%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/SasaTasks.cc.o
    [ 60%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/SeedingPositionTasks.cc.o
    [ 60%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/SetFaModeTasks.cc.o
    [ 63%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/rifdock_tasks/UtilTasks.cc.o
    [ 66%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/scaffold/Baseline9AScaffoldProvider.cc.o
    [ 66%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/scaffold/MorphingScaffoldProvider.cc.o
    [ 69%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/scaffold/NineAManager.cc.o
    [ 72%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/scaffold/SingleFileScaffoldProvider.cc.o
    [ 75%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/scaffold/util.cc.o
    [ 75%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/AnyPointTask.cc.o
    [ 78%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/RifDockResultTask.cc.o
    [ 81%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/SearchPointTask.cc.o
    [ 84%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/SearchPointWithRotsTask.cc.o
    [ 84%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/Task.cc.o
    [ 87%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/TaskProtocol.cc.o
    [ 90%] Building CXX object apps/rosetta/riflib/CMakeFiles/riflib.dir/task/util.cc.o
    [ 93%] Linking CXX shared library libriflib.so
    [ 93%] Built target riflib
    Scanning dependencies of target rif_dock_test
    [ 96%] Building CXX object apps/rosetta/CMakeFiles/rif_dock_test.dir/rif_dock_test.cc.o
    [100%] Linking CXX executable rif_dock_test
    [100%] Built target rif_dock_test
    [ 96%] Built target riflib
    Scanning dependencies of target rifgen
    [100%] Building CXX object apps/rosetta/CMakeFiles/rifgen.dir/rifgen.cc.o
    [100%] Linking CXX executable rifgen
    [100%] Built target rifgen
    [srgo@anode0 rifdock]$
    
##### (2) Do a Unit Test to check whether the build was successful.
     $ make test_libscheme
     
### 8. Running Rifdock
##### 1) The executable files for RifDock are built at:
    $ rifdock/build/apps/rosetta/rifgen
    $ rifdock/build/apps/rosetta/rif_dock_test
    
    [srgo@anode0 build]$ ls
    apps            CMakeFiles           external  schemelib
    CMakeCache.txt  cmake_install.cmake  Makefile
    [srgo@anode0 build]$ cd apps
    [srgo@anode0 apps]$ ls
    CMakeFiles  cmake_install.cmake  Makefile  rosetta
    [srgo@anode0 apps]$ cd rosetta
    [srgo@anode0 rosetta]$ ls
    CMakeFiles           Makefile  rif_dock_test  riflib
    cmake_install.cmake  python    rifgen

##### 2) The unit tests executable file is at:
    $ rifdock/build/schemelib/test/test_libscheme











#### Basic programming concepts
- compile: Change a file written in one language to another language
- build: a compiled version of a program
- bin: bin stands for the binary file which is a "non-text file". A Binary file is compiled file which could be directly understood and used by a computer.
- cxx: C++\
- gcc: GNU Compiler Collection includes front ends(like UI for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages (libstdc++,...). It is the basic compiler of Unix and Linux-based systems.
- front end: Front part of the system which is close to the user. The starting point or input part of the system. GUI, FEP, etc.
- back end: The part that supports the system on the backside like the database. Only accessible to programmers or administer.
- MPI: Massage Passing Interface is a standardized and implantable message-passing standard that is designed to function in paralleled computing architectures.
- Bash: Bourne Again Shell(BASH) is the free software that replaces Bourne shell. BASH is the basic UNIX shell of GNU, Linux, Mac OS X. Shell is the interface that is used when executing the command and program. Shell connects the kernel and user, it receives the command from the user and executes the program.
- UNIX: A multitasking, multiuser computer operating system(OS)
- Kernel: The core program of OS. It manages Security, Resource, Memory, and Abstraction of complicated information.
- CMake: The program which generates build files(such as Makefile) that build a specific project. It is not the building program itself, but it is a program to generate build files. CMakeList.txt --(CMake)--> Makefile --(make)--> Excutalbles 
In CMakeList.txt, there are two kinds of information. One is the minimum required version of CMake and another is the information about the project. For CMake has been changed from previous old ones(especially ver. 2.x), you need to identify whether the CMake which is installed in your computer is compatible to do its proper function.
- make: File Maganement Utility which execute compile based on the dependencies described in "Makefile"
- makefile: Setting file for "make" program, which could simplify the iterative compile process. It defines macro, targets, rules, commands, and dependancies to compile source files with make command.


#### Basic concepts to install program in linux
- / - root directory
- ./ - current directory
- ../ - superior directory
- ~ - home directory which is the directory that login user can use.
- ./program - run 'program' in current directory
- program - run 'program' in available address
- cp file1 /path/to/the/directory1 - cp file1 to directory1
- rm -rf file1 - Remove file1 without asking any questions.
- rm -rf directory1 - Remove directory1 and it all files under the directory without asking any questions.
- /usr - Univeral System Resources
- export PATH=$PATH:/path/to/the/file - add the specific address(/path/to/the/file) to $PATH 
- export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/the/file - add the specific address (of library) to $LD_LIBRARY_PATH
