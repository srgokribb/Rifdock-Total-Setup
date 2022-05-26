# RifDock Setup Protocol ver 1.0
- Setup protocol to build Rosetta and RifDock
- Tested on Red Hat 8.3.1 CentOS 8 
- Written by SeongRyeong Go on 28 Apr 2022.
- **If you want to directly see the method to build RifDock, see the step #7**

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
- Scaffolds
- Environment modules 4.1.4 (It is necessary to load other version of GCC.)
- GCC-6.5.0 (Necessary to build and run rifdock)

### Notice
- In this protocol, the command line which starts with **"#"** means **"root"** or **"administer"**, and the command line which starts with **"$"** means **"local user"**.

### Directory Structure
        
        home/   root/    usr/  

        /root/packages/
                        hdf/     boost-1.65.0/    

        /home/users/local_user/packages/
                         ninja-1.10.2/    re2c-1.0.3/   gcc-6.5.0/    gcc/

        /home/users/local_user/rosetta/
                 rosetta_src_release_3.9/     PyRosetta4/      rosetta_src_release_3.13/    rifdock/    scaffolds/    
                 psipred/     PatchDock/       ppi_tools/       silent_tools/      ncbi-blast/    ncbi_tools/

        /usr/local/
                 bin/   lib/  include/

- These are the major directories which are involved in setting up rifdock

### 1. Install HDF5 and Rosetta 3.13
#### 1) Install HDF5 (Ref: https://fossies.org/linux/hdf5/release_docs/INSTALL)
##### (1) Download HDF5 source file from The HDF5 Group server(https://www.hdfgroup.org/downloads/hdf5) . - Install ver 1.12.1.

##### (2) Login as administer and make the "packages" directory (This directory will be used to install new libraries, compilers, and etc).
    # mkdir packages
    # cd packages          // pwd is /root/packages/
     
##### (3) Move the downloaded file to the directory to install the file and unpack the tar file.             
    # tar -xvzf hdf5-1.12.1-centos8_64.tar.gz
- If you unpack the tar file, "hdf" directory will be generated.

##### (3) Enter in to the unpacked directory and execute installer file(.sh).
    # cd hdf
    # ./HDF5-1.12.1-Linux.sh
    There will be some explanation and notification for the lisence.
    Keep pressing "Enter" to scroll down.
    If you see the questions[y/n], type "y" and press "Enter".
    # ls
    # cd HDF_Group/HDF5/1.12.1
    # ls
    bin/ include/ lib/ share/
    
- You can see the new direictory is generated. At the lowest location of the directory, there are "bin", "include", "lib", "and "Share".
- "bin" directory: Executables
- "include" directory: Header files
- "lib" directory: Library files and linkers(.so)
- Caution: You need to check whether there are the files which have the same names with the HDF5 library files in the directory to set your files (/usr/local/bin /include /lib). If there are some files which could be overwitten, backup the files to other location and continue to the next step. It is important to keep your previous environment and avoid any fatal mistakes. 

##### (4) Copy the files to proper directory. /usr/local/bin/ user/local/include/ usr/local/lib/
    # cd bin
    # cp -r * /usr/local/bin
    # cd ../include
    # cp -r * /usr/local/include
    # cd ../lib
    # cp -r * /usr/local/lib

##### (5) (Optioanl) Refresh the shared library cache with "ldconfig".
    # ldconfig
- If your computer have other programs which refreshes the shared library cache, this step could be skipped.
- If there are some jobs which were already running and should be maintained, it is safer to execute this command after the jobs are finished.
- ldconfig command only could be executed by administer.

##### (6) Login as local user and set the PATH and LD_LIBRARY_PATH in .bashrc
    $ cd ~
    $ vi .bashrc
- Add the address of /usr/local/bin to PATH, and add /usr/local/include and /usr/local/lib to LD_LIBRARY_PATH.
    
            # .bashrc

            # Source global definitions
            if [ -f /etc/bashrc ]; then
                . /etc/bashrc
            fi

            # User specific environment
            PATH="$HOME/.local/bin:$HOME/bin:$PATH"
            export PATH
            PATH=$PATH:/usr/local/bin
            export PATH

            LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib:/usr/local/include
            export LD_LIBRARY_PATH


            # Uncomment the following line if you don't like systemctl's auto-paging feature:
            # export SYSTEMD_PAGER=

            # User specific aliases and functions

##### (7) Apply the changed environment.
    $ source .bashrc
    
- After you execute this command, close the terminal and open a new terminal to continue.

##### (8) Check wether $PATH and $LD_LIBRARY_PATH are updated.
    $ echo $PATH
    $ echo $LD_LIBRARY_PATH
- If PATH is updated correctly, you can proceed to rosetta install and compilation.
- Do not add multiple versions of hdf5 to PATH or LD_LIBRARY_PATH. It will make a fatal error.
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

##### **I tried to use recent rosetta compiled with HDF5, but I used rosetta compiled with mpi because of some problems with zlib and our server environment.**

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
- PSIPRED Version 4.0 By David Jones, January 2016
##### (1) Download PsiPred source code from Git-hub pirspred/pirspred (https://github.com/psipred/psipred).
- The file name will be "psipred-master.zip".
##### (2) Move the zip file to the directory to install psipred in the Linux computer.

##### (3) Unpack the zip file by
    $ unzip psipred-master.zip
    $ mv psipred-master PSIPRED

##### (4) Compile and install psipred.
    $ cd PSIPRED
    $ cd src
    $ make
    $ make install
    $ export PATH=$PATH:/path/to/psipred
- If the installation is finished, executable "psipred" will be in bin directory
- You can use "runpsipred" to run psipred program.

##### (5) Test whether the psipred is run
    $ cd example
    $ runpsipred example.fasta
- If you didn't install PSI-BLAST, there will be the error that cannot find .blast files to run the psipred.
- However, Rifdock only requires "runpsipred_single" executable file, so there could be no need to install PSI-Blast, NCBI Toolkit, UNIREF90 and etc. 
- So, Step 5.2) to 5.6) would be unnecessary.

        #### 2) Install PSI-BLAST
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

        ##### 3) Install NCBI Software Development Toolkit
        - This package can install NCBI Blast and Impala which are needed to run psipred.

        ##### (1) Download NCBI software development toolkit from NCBI FTP server (https://ftp.ncbi.nih.gov/toolbox/ncbi_tools/)
        - ncbi.tar.gz

        ##### (2) Unpack tar file and change the name of the directory
            $ mkdir NCBI_TOOLKIT
            $ cd NCBI_TOOLKIT
            $ tar -xvzf ncbi.tar.gz
        - the unpacked directory name will be "ncbi". In order to install NCBI Toolkit, you shouldn't change the name of this directory.

        ##### (3) Check the guide in the README file.
            $ vi README

        ##### (4) In order to build NCBI toolkit for Linux, see make/readme.unx
            $ vi make/readme.unx

        ##### (5) Install NCBI ToolKit
            $ ./ncbi/make/makedis.csh
        - During installation, the program cannot find -ltspi , -lunistring, -lidn2 and the install was stopped because of the error.

                /usr/bin/ld: cannot find -ltspi
                /usr/bin/ld: cannot find -lunistring
                /usr/bin/ld: cannot find -lidn2
                collect2: error: ld returned 1 exit status
                make: *** [makenet.unx:1268: idfetch] Error 1
                FAILURE primary make status = 0, demo = 0, threaded_demo = 0, net = 2
                #######
                #        #####   #####    ####   #####
                #        #    #  #    #  #    #  #    #
                #####    #    #  #    #  #    #  #    #
                #        #####   #####   #    #  #####
                #        #   #   #   #   #    #  #   #
                #######  #    #  #    #   ####   #    #

        #### 4) Install Impala from NCBI development tool kit 
        - Even though the README file in psipred said that Impala is need to run psipred, there is no file README.imp in ncbi/doc. So, it is unable to install impala currenty.

        #### 5) Install Sequence Data Bank (UNIREF90)
        ##### (1) Download UNIREF90 from Uniprot DB server https://ftp.uniprot.org/pub/databases/uniprot/uniref/uniref90/).
        - The size of uniref90.fasta.gz is 32.0G and that of uniref90.xml.gz is 49.1G. So, it will take a few hours to download the files.

        ##### (2) make uniref90 directory and unpack gz files.
            $ mkdir uniref90
            $ cd uniref90
            $ gzip -d uniref90.fasta.gz
            $ gzip -d uniref90.xml.gz

        #### 6) Run psipred
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
            
### 6. Set the environment to install and run rifdock (Environment modules, gcc-6.5.0)
#### 1) Install Environment modules
##### (1) Check whether your computer have Environment Modules.
- **[Caution] Before you install Environment modules, it is critical to check whether there is already installed Environment modules. If you install additional module, the previous path settings of modules will be corrupted and it will make fatal problem.**
- You can check whether you have moduels by below commands.
    $ module --version
    $ which module
    $ whereis Module

##### (2) Download source files of the modules (https://sourceforge.net/projects/modules/files/Modules/modules-4.1.4/).

##### (3) Install Environment modules (Ref: https://www.admin-magazine.com/HPC/Articles/Environment-Modules).
    # cd /usr/local
    # mkdir Modules
    # mkdir src
    # cp modules-4.1.4.tar.gz /usr/local/Modules/src
    # tar -xvzf modules-4.1.4.tar.gz
    # cd modules-4.1.4
    # ./configure
    # make
    # make install

#### 2) Install GCC 6.5.0
##### (1) Current GCC verison could be identified by using
     $ gcc --version
- The version of other compilers also could be identified by using '--version'.

##### (2) Check the kernerl version of your system.
     $ uname -r
- You need to check wether gcc-6.5.0 is compatible with the kernel version of your computer.

##### (3) Download GCC 6.5.0 from GNU server(https://ftp.gnu.org/gnu/gcc/gcc-6.5.0/). 
##### (4) Move the tar.gz file to rosetta folder and unpack the file..
     $ tar -xvzf gcc-6.5.0.tar.gz
     
##### (5) Install GCC-6.5.0. on the local directory (Ref: https://gcc.gnu.org/wiki/InstallingGCC).
     $ cd gcc-6.5.0
     $ ./contrib/download_prerequisites
     $ cd ..
     $ mkdir objdir
     $ cd objdir
     $../configure --prefix=$HOME/GCC-6.5.0 --enable-languages=c,c++,fortran --disable-multilib
     $ make
     $ make install
     $ cd ~ && vi .bashrc
 - I used "$../configure --prefix=$/home/users/srgo/packages/gcc --enable-languages=c,c++,fortran --diable-multilib" to install gcc in local directory.
 
##### (6) Add the path to gcc/lib64 to LD_LIBRAY_PATH.
     LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/users/srgo/packages/gcc/lib64
     export LD_LIBRARY_PATH
     
 - In later stage, "CXX=/path/to/gcc/bin/g++ CC=/path/to/gcc/bin/gcc" option will be used to build rifdock.

##### (7) Make "modulefile" for new gcc in /user/share/Modules/modulefiles.
     # cd /usr/share/Modules/modulefiles              //Go to the directory where Modules is installed.
     # mkdir gcc
     # cd gcc
     # vi 6.5.0                                       //This file have basically same format of "modules" file in modulefiles directory.
     
        #%Module 1.0
        #set    BASEDIR         /opt/[module-info name]
        set     BASEDIR         /path/to/new/gcc
        prepend-path    PATH            ${BASEDIR}/bin
        prepend-path    CPATH           ${BASEDIR}/include
        prepend-path    MANPATH         ${BASEDIR}/share/man
        prepend-path    LD_LIBRARY_PATH ${BASEDIR}/lib64

     # module avail                             //If you check the available modules, then you can see the modulefile of gcc is updated.
     gcc/6.5.0
     
##### (8) Load new gcc using environment modules.
     $ module load gcc/6.5.0
     
    If you want to automatically load gcc-6.5.0, open .bashrc and add "module load gcc/6.5.0" at the end of the file.
    
     $ vi .bashrc
     
        if [ -f /etc/bashrc ]; then
                . /etc/bashrc
        fi

        # User specific environment
        PATH="$HOME/.local/bin:$HOME/bin:$PATH"
        export PATH
        PATH=$PATH:/home/users/srgo/packages/re2c-1.0.3/build:/home/users/srgo/packages/ninja-1.10.2
        export PATH

        LD_LIBRARY_PATH=/home/users/srgo/packages/gcc/lib64:/opt/ompi/4.1.0-gt/lib:/opt/intel/mkl/8.1.1/lib/em64t:/opt/intel/mkl/8.1.1/lib/em64t
        export LD_LIBRARY_PATH

        # Uncomment the following line if you don't like systemctl's auto-paging feature:
        # export SYSTEMD_PAGER=

        # User specific aliases and functions
        module load gcc/6.5.0
- If you don't want to load gcc-6.5.0, just add "#" infront of "module load gcc/6.5.0" and inactivate it.
- You need to load gcc-6.5.0 when you build or run rifdock.
 
### 7. Install Boost and Ninja to build RifDock
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
    
##### (3) Add the path to executable re2c
    $ export PATH=$PATH:/path/to/re2c-1.0.3/build
- Add the path to the dirctory where "re2c" executable is installed to $PATH.

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

### 8. Build Rosetta cxx11_omp and RifDock
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
- When later version of Boost(Boost-1.74.0 or Boost-1.78.0) was used instead of Boost-1.65.0, it made BOOST_BITMASK error during "make" step.

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
     
### 9. Running Rifdock
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
##### ※ Step #7 and #8 were tested more than two times.
 

### 10. Download misc(cao_2021_protocol, scilent_tools, ppi_tools, scaffolds) for RifDock
#### 1) Download cao_2021_protocol
##### (1) Download design scripts and main pdb files from http://files.ipd.uw.edu/pub/robust_de_novo_design_minibinders_2021/supplemental_files/scripts_and_main_pdbs.tar.gz
##### (2) Move the file to 'resources' directory and unpack the tar file.
    $ cd resources
    $ tar -xvzf scripts_and_main_pdbs.tar.gz
    
##### (3) Copy "cao_2021_protocl" directory to 'rosetta' directory.
    $ cd scripts_and_main_pdbs/supplemental_files/
    $ cp -r cao_2021_protocol/ ../../../rosetta
  
##### (4) Add the PATH to cao_2021_protocol to .bashrc (This will be covered in step #10).

#### 2) Download scilent_tools and ppi_tools
##### (1) scilent_tools and ppi_tools are in cao_2021_protocol/github_backup. So, copy them to rosetta directory.
    $ cd cao_2021_protocol/github_backup/
    $ cp -r scilent_tools ../../
    $ cp -r ppi_tools ../../
- Currnt working directory is $HOME/rosetta/cao_2021_protocol/github_backup/

##### (2) Add the PATH to scilent_toosl and ppi_tools to .bashrc (This will be covered in step #10).

#### 3) Download Miniprotein Scaffold Library
##### (1) Download all the scaffolds from http://files.ipd.uw.edu/pub/robust_de_novo_design_minibinders_2021/supplemental_files/scaffolds.tar.gz
##### (2) Move the file to 'resources' directory and unpack the tar file.
    $ cd resources
    $ tar -xvzf scaffolds.tar.gz
- If you unpack the tar file, it will generate "supplemental_files" directory.

##### (3) Copy scaffold directory to rosetta directory.
    $ cp -r supplemental_files/scaffolds ../rosetta

##### (4) Add the PATH scaffold directory to .bashrc (This will be covered in step #10).


### 11. Set the proper PATH to all binaries
##### (1) Move to $HOME and open .bashrc
    $cd ~
    $vi .bashrc
- It is strongly recommended to copy the .bashrc to backup directory for safe.

##### (2) Set the cusor in the empty line and change to "insert mode" by press "Shift + S" buttons and add the PATH.

from 

        # .bashrc

        # Source global definitions
        if [ -f /etc/bashrc ]; then
                . /etc/bashrc
        fi

        # User specific environment
        PATH="$HOME/.local/bin:$HOME/bin:$PATH"


        # Uncomment the following line if you don't like systemctl's auto-paging feature:
        # export SYSTEMD_PAGER=

        # User specific aliases and functions


to

        # .bashrc

        # Source global definitions
        if [ -f /etc/bashrc ]; then
                . /etc/bashrc
        fi

        # User specific environment
        PATH="$HOME/.local/bin:$HOME/bin:$PATH"
        export PATH
        PATH=$PATH:/home/users/srgo/rosetta/rosetta_src_2021.16.61629_bundle/main/source/bin:/home/users/srgo/rosetta/rifdock/build/apps/rosetta:/home/users/srgo/rosetta/scaffolds:/home/users/srgo/rosetta/cao_2021_protocol:/home/users/srgo/rosetta/psipred:/home/users/srgo/rosetta/psipred/bin:/home/users/srgo/rosetta/PatchDock:/home/users/srgo/rosetta/silent_tools:/home/users/srgo/rosetta/NCBI/BLAST/bin
        export PATH

        LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
        export LD_LIBRARY_PATH


        # Uncomment the following line if you don't like systemctl's auto-paging feature:
        # export SYSTEMD_PAGER=

        # User specific aliases and functions

##### (3) If you finished the revision, press "ESC" button to exit from "insert mode" and press ":" button and type "wq" and press "Enter" button to save the changes and exit from the file.
        :wq
        (Enter)

##### (4) Refresh the PATH by "source .bashrc" command and check wether the PATH is sucessfully added.
    $ source .bashrc
    $ echo $PATH
    
### 12. Try rifdock_tutorial
#### I'm still working to follow the cao's protocol who made rifgen and rifdock. 
#### I will update the entire process when I stably setup and finiched to follow the protocol.

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
- cp file1 /path/to/the/directory1 - Copy file1 to directory1
- cp -r direcroty1 /path/to/the/directory2 - Copy direcroty1 and it's all of subdirectories to direcroty 2 
- rm -rf file1 - Remove file1 without asking any questions.
- rm -rf directory1 - Remove directory1 and it all files under the directory without asking any questions.
- /usr - Univeral System Resources Basically most binary files and libraries are installed in here.
- export PATH=$PATH:/path/to/the/file - add the specific address(/path/to/the/file) to $PATH 
- export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/the/file - add the specific address (of library) to $LD_LIBRARY_PATH
- ldconfig: Refreshes shared library cache. It is needed to be used after changing LD_LIBRARY_PATH or adding .conf file to /etc/ld.so.conf.d/
- pkg-config: The software which can search specific library which is needed to compile a program from the source code of the program. It uses "x.pc" file which contains the details about the library "x".
