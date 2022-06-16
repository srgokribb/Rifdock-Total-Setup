# RifDock Setup Protocol ver 1.2
- Setup protocol to build Rosetta and RifDock
- Tested on Red Hat 8.3.1 CentOS 8
- GCC-6.5.0 loaded using Environment modules to build Rifdock correctly.
- Written by SeongRyeong Go on 28 Apr 2022.
- Last revised on 17 June 2022.
- **If you want to directly see the method to build RifDock, see the step #8**
- **Basically I'm a beginner to Linux. So, there could be some mistakes that I missed.**
- I tried to made this mannual as easy as possible, so that anyone, who patiently follows the mannual, could successfully install rifdock.
- **Specially give thanks to Longxing Cao and Brian Coventry, who are the members of David Baker's group in UW, for sharing Rifdock.**

## Least spec to run rifdock
- Recommended by authors of Rifdock: 32+ core CPU, 16-80GB RAM depending on the size of the target
- As for installation, ~10 cpus were enough.
- As for run, I sensibly recommned 50 core CPU or more, 200GB RAM or more **per one target**.

### These are the required programs or resources to run RifDock
**(Bolded programs are directly necessary to build Rifdock)**
- Rosetta_3.13 (Recent Rosetta later than 2020, HDF5 is needed)
- DAIphaBall(Part of Rosetta)
- PyRosetta4
- PatchDock
- PsiPred
- Environment modules 4.5.2 (It is necessary to load other version of GCC.)
- GCC-6.5.0 (Necessary to build and run rifdock)
- **Rosetta_3.9 compiled as cxx11_omp (Re2c and Ninja are needed)**
- **RifDock (Boost 1.65.0 is needed)**
- silent_tools
- Scaffolds
- ss_grouped_vall_all.h5(optional)


### Notice
- In this protocol, the command line which starts with **"#"** means **"root"** or **"administer"**, and the command line which starts with **"$"** means **"local user"**.

### Directory Structure
        
        home/   root/    usr/   opt/

        /root/packages/
                        hdf/     boost-1.65.0/    

        /home/users/local_user/packages/
                         ninja-1.10.2/    re2c-1.0.3/   gcc-6.5.0/    gcc/

        /home/users/local_user/rosetta/
                 rosetta_src_release_3.9/     PyRosetta4/      rosetta_src_release_3.13/    rifdock/    scaffolds/    
                 psipred/     PatchDock/       ppi_tools/       silent_tools/

        /usr/local/
                 bin/   lib/  include/  share/
        
        /usr/share/Modules/
                        modulefiles/gcc/6.5.0    <-- This "6.5.0" file is the modules file which could be loaded by Environment modules
                        
- These are the major directories which are involved in setting up rifdock

## 1. Install HDF5 and Compile Rosetta_3.13 with HDF5
### 1) Install HDF5 (Ref: https://fossies.org/linux/hdf5/release_docs/INSTALL)
- **Caution: Do not install multiple versions of hdf5 under usr/local. If multiple versions of same compiler are accessible to $PATH or $LD_LIBRARY_PATH, it could make fatal error.** If you are an user of the server and new to linux, be cautious and ask a help to the system manager or expert.

#### (1) Download HDF5 source file from The HDF5 Group server(https://www.hdfgroup.org/downloads/hdf5) . - Install ver 1.12.1.

#### (2) Login as administer and make the "packages" directory (This directory will be used to install new libraries, compilers, and etc).
    # mkdir packages
    # cd packages          // pwd is /root/packages/
    
- This is just an example, but I recommned you to make your own directory to install compilers or programs.

#### (3) Move the downloaded file to the directory you've made (packages/) and unpack the tar file.             
    # tar -xvzf hdf5-1.12.1-centos8_64.tar.gz
    
- If you unpack the tar file, "hdf" directory will be generated.

#### (3) Enter in to the unpacked directory and execute installer file(.sh).
    # cd hdf
    # ./HDF5-1.12.1-Linux.sh
    There will be some explanation and notification for the lisence.
    Keep pressing "Enter" to scroll down.
    If you see the questions[y/n], type "y" and press "Enter".
    # ls
    # cd HDF_Group/HDF5/1.12.1
    # ls
    bin/ include/ lib/ share/
    
- You can see the new direictory named as "hdf" is generated. At the lowest location of the directory, there are "bin", "include", "lib", "and "Share".
- "bin" directory: Executables
- "include" directory: Header files
- "lib" directory: Library files and linkers(.so)
- **Caution: You need to check whether there are the files which have the same names with the HDF5 library files in the directory to finally install your files (/usr/local/bin /include /lib). If there are some files which could be overwitten, backup the previous files to the other directory and continue to the next step. It is important not to destroy your previous environments and avoid any fatal mistakes.** 

#### (4) Copy the files to proper directory. /usr/local/bin/ user/local/include/ usr/local/lib/
    # cd bin
    # cp -r * /usr/local/bin
    # cd ../include
    # cp -r * /usr/local/include
    # cd ../lib
    # cp -r * /usr/local/lib

#### (5) (Optioanl) Refresh the shared library cache with "ldconfig".
    # ldconfig
- If your computer have the system which refreshes the shared library cache, this step could be skipped.
- If there are some jobs which were already running and should be maintained, it is safer to execute this command after the jobs are finished.
- ldconfig command only could be executed by administer.

#### (6) Login as local user and set the PATH and LD_LIBRARY_PATH in .bashrc
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

#### (7) Apply the changed environment.
    $ source .bashrc
    
- After you execute this command, close the terminal and open a new terminal to continue.

#### (8) Check whether $PATH and $LD_LIBRARY_PATH are updated.
    $ echo $PATH
    $ echo $LD_LIBRARY_PATH
    
- If PATH is updated correctly, you can proceed to rosetta install and compilation.
- Because the addresses are added to ".bashrc", it will be maintained after you close your terminal or open a new terminal. If you don't need hdf5 anymore, remove the address to hdf5 binary and library from ".bashrc".

### 2) Install Rosetta 3.13 and compile with HDF5
- Check whether the version of compilers in your machine and environment supports Rosetta before the installation (https://new.rosettacommons.org/docs/latest/build_documentation/Cxx11Support).
- GCC/g++: Version 4.8 or later (https://gcc.gnu.org/releases.html)
- Clang/llvm on Linux: Version 3.3 or later (https://releases.llvm.org/download.html)

#### (1) Go to Rosetta commons and apply for academic liscence (https://www.rosettacommons.org/software/license-and-download). 

#### (2) After login in with the ID and password of the license, go to 'Downloads' and enter Rosetta 3.13 - Download Rosetta 3.13. 

#### (3) Download Rosetta 3.13 source (5.2G) file (rosetta_src_3.13_bundle.tgz) and copy the source file to the folder on the Linux computer where the Rosetta 3.13 will be installed (https://www.rosettacommons.org/downloads/academic/3.13/). 

#### (4) Unpack the tar file.
     $ tar -xvzf rosetta_src_3.13_bundle.tgz
- The installation takes 20~30 min and rosetta_scr_release_bundle folder will be generated.

#### (5) Move to 'source' folder.
    $ cd rosetta_src_release_bundle/main/source

#### (6) Check 'scons.py' is in the source folder. This file is the software needed to compile Rosetta (already included in the rosetta bundle in main/source folder). 

#### (7) Compile rosetta in HDF5 format which is the basic form of Rosetta.
    $ ./scons.py -j10 mode=release extras=hdf5 rosetta_scripts

- '-j 10' means using 10 cores of CPU for compiling 
- 'mode=release' means compile with optimizations to produce a faster version of rosetta
- 'mode=debug' or not mentioning any mode include additional checks which slow down Rosetta runs (not recommended).
- 'extra=hdf5' option is used to build basic rosetta. If you use this option, add 'rosetta_scripts' at the end of the line.
- 'extras=static' builds static binaries which could be useful to copy or run the apps on other systems.
- 'extras=mpi' compiles Rosetta in Massage Passing Interface format which supports MPI run (Recommended for the cluster system like HPC)
- If the extra option is mpi, use "bin/rosetta_scripts.mpi.linuxgccrelease" instead of "rosetta_scripts". If you chose to build as static, then use "bin/rosetta_scripts.default.linuxgccrelease".
- There are additional option to compile with specific cxx version. In that case, use the form like "./scons.py -j 10 mode=release bin cxx=clang cxx_ver=4.5".
- The compiling process will take about 1h. The time could be reduced by increasing the number of CPUs ('-j 20' or more)

#### (8) Check the compiled result. If the compilation was successfully done, the last sentence of the terminal shows the result like below.
    "Install file: "build/src/release/linux/4.18/64/x86/gcc/8/default/rosetta_scripts.default.linuxgccrelease" as "bin/rosetta_scripts.default.linuxgccrelease"
    scons: done building targets.
    
#### (9) The executable rosetta which is named as "rosetta_scripts.hdf5.linuxgccrelease" will be in /main/source/bin.

## 2. Install DAIphaBall
#### (1) DAIphaBall is a part of Rosetta. Move to rosetta_src_release/main/source/external/DAIphaBall.

#### (2) Type "make" to build DAIphaBall.
    $ make
- DAlphaBall.gcc will be made.

## 3. Install PyRosetta
#### (1) Get the license for PyRosetta in Rosetta Commons(https://www.rosettacommons.org/software/license-and-download)

#### (2) Check the Python version installed on the Linux computer.
    $ python --version
    
#### (3) Download proper version of PyRosetta4 in PyRosetta server(https://graylab.jhu.edu/download/PyRosetta4/archive/release/)

- Release : Speed optimized build for production runs.
- MinSizeRel : Build optimized to reduce memory which could be suitable for low memory systems.
- Debug : Binaries compiled in debug mode with additional asserts enabled and with debug-info compiled in. Build for debugging
- Python-x.x versions : The builds of PyRosetta for the specific Python version.
- "PyRosetta4.Release.python36.linux.release-316.tar.bz2" is selected for installation.

#### (4) Move the downloaded file to the directory to install PyRosetta and unzip the file.
    $ tar -vjxf PyRosetta4.Release.python36.linux.release-316.tar.bz2
    
#### (5) Enter to PyRosetta directory and type
    $ cd setup
    $ python setup.py install
    
- **This step requires administrative access. You need to sign in as administer or "root" account.**
#### (6) Check whether the PyRosetta is successfully installed. If it was successful, you can see these sentences in the last sentence of the installation.
    Installed /usr/local/lib/python3.6/site-package/pyrosetta-2022.14+release.d95c942-py3.6-linux-x86_64.egg
    Processing dependencies for pyrosetta==2022.14+release.d95c942
    Finished processing dependencies for pyrosetta==2022.14+release.d95c942

## 4. Install PatchDock
##### (1) Download PatchDock from PatchDock server(https://bioinfo3d.cs.tau.ac.il/PatchDock/).

##### (2) Unzip the file by
    $ unzip patch_dock_download.zip

##### (3) PatchDock do not require further install process. Read 'README.md' to run PatchDock.


## 5. Install PsiPred
- PSIPRED Version 4.0 By David Jones, January 2016
#### (1) Download PsiPred source code from Git-hub pirspred/pirspred (https://github.com/psipred/psipred).
- The file name will be "psipred-master.zip".
#### (2) Move the zip file to the directory to install psipred in the Linux computer.

#### (3) Unpack the zip file by
    $ unzip psipred-master.zip
    $ mv psipred-master pripred

#### (4) Install psipred.
    $ cd psipred
    $ cd src
    $ make
    $ make install
    $ export PATH=$PATH:/path/to/psipred
- If the installation is finished, executable "psipred" will be in bin directory
- You can use "runpsipred_single" to run psipred program.

#### (5) Open "runpsipre_single" and revise the path to the "execdir" and "datadir".
    $ cd psipred
    $ vi runpsipred_single
    
        #!/bin/tcsh

        # This is a simple script which will carry out all of the basic steps
        # required to make a PSIPRED prediction. Note that it assumes that the
        # following programs are available in the appropriate directories:
        # seq2mtx - PSIPRED V4 program
        # psipred - PSIPRED V4 program
        # psipass2 - PSIPRED V4 program

        # NOTE: Script modified to be more cluster friendly (DTJ April 2008)

        # Where the PSIPRED V4 programs have been installed
        set execdir = ./bin                                               // Change here to /path/to/psipred/bin

        # Where the PSIPRED V4 data files have been  installed
        set datadir = ./data                                              // Change here to /path/to/psipred/data

#### (6) Test whether the psipred is run
    $ cd example
    $ ../runpsipred_single example.fasta
    
- Rifdock protocol only requires "runpsipred_single" executable file, so there is no need to install PSI-Blast, NCBI Toolkit, UNIREF90 and etc. 
            
## 6. Set the environment to install and run rifdock (Environment modules, gcc-6.5.0)
### 1) Install Environment modules
#### (1) Check whether your computer have Environment Modules.
- **[Caution] Before you install Environment modules, it is critical to check whether there is already installed Environment modules. If you install additional module, the previous path settings of modules will be corrupted and it will cause a lot of fatal problems.**
- You can check whether you have moduels by below commands.
####
    $ module --version
    $ which module
    $ whereis Module

#### (2) Download source files of the modules (https://sourceforge.net/projects/modules/files/Modules/modules-4.5.2/).

#### (3) Install Environment modules (Ref: https://www.admin-magazine.com/HPC/Articles/Environment-Modules).
    # cd /usr/local
    # mkdir Modules
    # mkdir src
    # cp modules-4.5.2.tar.gz /usr/local/Modules/src
    # tar -xvzf modules-4.5.2.tar.gz
    # cd modules-4.5.2
    # ./configure
    # make
    # make install

### 2) Install GCC 6.5.0
#### (1) Current GCC verison could be identified by using
     $ gcc --version
- The version of other compilers also could be identified by using '--version'.

#### (2) Check the kernerl version of your system.
     $ uname -r
- You need to check whether gcc-6.5.0 is compatible with the kernel version of your computer.

#### (3) Download GCC 6.5.0 from GNU server(https://ftp.gnu.org/gnu/gcc/gcc-6.5.0/). 
#### (4) Move the tar.gz file to rosetta folder and unpack the file..
     $ tar -xvzf gcc-6.5.0.tar.gz
     
#### (5) Install GCC-6.5.0. on the local directory (Ref: https://gcc.gnu.org/wiki/InstallingGCC).
     $ cd gcc-6.5.0
     $ ./contrib/download_prerequisites
     $ cd ..
     $ mkdir objdir
     $ cd objdir
     $../configure --prefix=$HOME/path/to/make/GCC-6.5.0 --enable-languages=c,c++,fortran --disable-multilib
     $ make
     $ make install
     $ cd ~ && vi .bashrc
- "--disable-multilib" is used not to make libraries for 32-bit, bacause my linux environment is 64-bit. Check whether the options are fit to your environment.
 
#### (6) Make "modulefile" for new gcc in /user/share/Modules/modulefiles.
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
     
#### (7) Login as local user and load new gcc using environment modules.
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

        # User specific aliases and functions
        module load gcc/6.5.0
        
- If you don't want to load gcc-6.5.0, just add "#" infront of "module load gcc/6.5.0" and inactivate it.
- You need to load gcc-6.5.0 when you build or run rifdock.
- If you want to make other users to accessible to gcc-6.5.0, you need to change the authority of your directory by using "chmod". Making the system which load gcc-6.5.0 from /opt/gcc/gcc-6.5.0 could be another option (I used this method, but try to build on your local directory first).

#### (8) Check whether the current GCC version is changed to gcc-6.5.0
    $ which gcc
    ~/path/to/gcc-6.5.0/bin/gcc

## 7. Install Boost and Ninja to build RifDock
- To build RifDock, obtain a copy of gcc with version >= 5.0
- Install Boost version 1.65 or later
- Tested CentOS8(Red Hat gcc 8.3.1) with Boost-1.65.0

### 1) Install Boost
**Notification: I recommend you to log in as "root" or "administer" to the properly install boost.**

#### (1) Download Boost version 1.65.0 from Boost C++ Library server(https://www.boost.org/users/history/version_1_65_0.html). 
- Latest version of Boost (1.74.0 or 1.78.0) was not compatible with Rifdock.

#### (2) Move the downloaded file to the folder to install and unpack the file (Ref: https://valelab4.ucsf.edu/svn/3rdpartypublic/boost/more/getting_started/windows.html).
     # cd /root/
     # mkdir packages
     # cd packages
     # tar -xvzf boost_1_65_0.tar.gz
     # cd boost_1_65_0 

#### (3) Select the location to install the boost by using
     # ./bootstrap.sh --prefix=/path/to/install/boost/
- **If you are sign in as "root" or administer and want to make them automatically added to /usr/local directory, you don't need to add --prefix option. Check whether libboost_xxx.so and libboost_xxx.so.1.65.0 files are generated in /usr/local/lib and /usr/local/include.**

#### (4) Start installation
     # ./b2 install
- The installation process will take a quite long time. 
- Even though there could be some minor errors like "warning: unnecessary parentheses in the declaration of 'assert_mot_arg' [-Wparentheses]", it didn't make any error during compiling Rifdock.
- **Caution: If multiple versions of boost are installed, it makes an error when compiling RifDock. So, if you want to use another version of boost, it is recommended to delete the previous boost (check /usr/local/include /usr/local/lib /usr/lib) and install a new one.**

### 2) Load gcc-6.5.0 by modules.
#### - **Important: From here to finishing the building rifdock, keep gcc-6.5.0 mounted on the terminals. See 6.2)-(8)**
    $ module load gcc/6.5.0
    $ which gcc
    ~/packages/gcc/bin/gcc
- If you don't compile rifdock using GCC 5.x, 6.x, or 7.x, you can not install really working rifdock.
- You can install rifdock eventhough you do not change GCC version, you will eventually get "ASSERTION ERROR" when you try to use rifgen, one of the major executable in Rifdock.

### 3) Install re2c - Needed to run Ninja
#### (1) Download re2c from downloading site (https://opensuse.pkgs.org/15.3/opensuse-oss-x86_64/re2c-1.0.3-1.18.aarch64.rpm.html).
#### (2) Install re2c by using following commands (Ref: https://re2c.org/build/build.html).
    $ tar -xvzf re2c-1.0.3.tar.gz
    $ cd re2c-1.0.3
    $ mkdir build
    $ cd build
    $ ../configure && make && make install
    
#### (3) Add the path to executable re2c
    $ export PATH=$PATH:/path/to/re2c-1.0.3/build
- Add the path to the dirctory where "re2c" executable is installed to $PATH.

### 4) Install Ninja
#### - In order to build rosetta cxx11_omp, Ninja shold be installed and proper PATH of Ninja need to be set.
#### (1) Download Ninja 1.10.2 from Git-hub(https://github.com/ninja-build/ninja/releases). (Ref:https://github.com/ninja-build/ninja/wiki, https://github.com/ninja-build/ninja) 
 
#### (2) Install Ninja
     $ tar xvzf- ninja-1.10.2.tar.gz
     $ cd ninja-1.10.2
     $ ./configure.py --bootstrap
     
#### (3) Set the proper PATH of Ninja (Necessary)
     $ echo $PATH
     $ export PATH=$PATH:/home/users/local/ninja-1.10.2/
     $ echo $PATH
- You can add new PATH by using "export PATH=$PATH:/path/to/directory"

## 8. Build Rosetta cxx11_omp and RifDock
- Rifdock is made based on OpenMP(OMP) which is the compiler could make single thread program to multithread program.
- Because Rifdock requires some libraries and resources in Rosetta, Rosetta also need to be build with OMP.

### 1) Build Rosetta 3.9 as cxx11_omp using Ninja
#### (1) Download Rosetta 3.9 source (2.8G) file (rosetta_src_3.9_bundle.tgz) and copy the source file to the directory where the Rosetta 3.9 will be installed. 

#### (2) Unpack Rosetta
     $ tar -xvzf rosetta_src_3.9_bundle.tgz

#### (3) Move to main/source folder
     $ cd rosetta_src_release_bundle/main/source
     
#### (4) build rosetta cxx11_omp using Ninja(Ref: See 'Extra tools' in https://ninja-build.org/manual.html)
     $ cd rosetta_src_release_bundle/main/source
     $ CXX=/path/to/gcc-6.5.0/bin/g++ CC=/path/to/gcc-6.5.0/bin/gcc ./ninja_build.py cxx11_omp -t rosetta_scripts -remake

### 2) Build RifDock
#### (1) Copy rifdock repository and build Rifdock (https://github.com/rifdock/rifdock)
     $ unzip rifdock-master.zip
     $ mv rifdock-master rifdock
     $ cd rifdock
     $ mkdir build
     $ cd build
     $ CXX=/path/to/gcc-6.5.0/bin/g++ CC=/path/to/gcc-6.5.0/bin/gcc CMAKE_ROSETTA_PATH=/Path/to/a/rosetta/main CMAKE_FINAL_ROSETTA_PATH=/Path/to/a/rosetta/main/source/cmake/build_cxx11_omp cmake .. -DCMAKE_BUILD_TYPE=Release
     $ make -j10 rif_dock_test rifgen
- When later version of Boost(Boost-1.74.0 or Boost-1.78.0) was used to build rifdock, instead of Boost-1.65.0, it caused BOOST_BITMASK error during "make" step.

### 2) The unit tests executable file is at:
    $ rifdock/build/schemelib/test/test_libscheme
 

## 10. Download misc(cao_2021_protocol, scilent_tools, ppi_tools, scaffolds) for RifDock
### 1) Download cao_2021_protocol
#### (1) Download design scripts and main pdb files from http://files.ipd.uw.edu/pub/robust_de_novo_design_minibinders_2021/supplemental_files/scripts_and_main_pdbs.tar.gz
#### (2) Move the file to 'resources' directory and unpack the tar file.
    $ cd resources
    $ tar -xvzf scripts_and_main_pdbs.tar.gz
    
#### (3) Check the files in cao_2021_protocol.
    $ cd scripts_and_main_pdbs/supplemental_files/
- There will be cao_2021_protocol_guide.txt which is the practical guide to run rifdock.
- There will be .xml, .py, .flags, .sh, .ipynb and etc. The usage of the files are described in cao_2021_protocol_guide.txt.
  
### 2) Download scilent_tools and ppi_tools
#### (1) scilent_tools and ppi_tools are in cao_2021_protocol/github_backup.
    $ cd cao_2021_protocol/github_backup/

#### (2) Add the PATH to scilent_tools to .bashrc (This will be covered in step #11).

### 3) Download Miniprotein Scaffold Library
#### (1) Download all the scaffolds from http://files.ipd.uw.edu/pub/robust_de_novo_design_minibinders_2021/supplemental_files/scaffolds.tar.gz
#### (2) Move the file to 'resources' directory and unpack the tar file.
    $ cd resources
    $ tar -xvzf scaffolds.tar.gz
- If you unpack the tar file, it will generate "supplemental_files" directory.

#### (3) Copy scaffold directory to rosetta directory.
    $ cp -r supplemental_files/scaffolds ../rosetta

#### (4) Add the PATH scaffold directory to .bashrc (This will be covered in step #11).

### 4) Download modular repeat protein database
#### (1) Download ss_grouped_vall_all.h5 (files.ipd.uw.edu/pub/modular_repeat_protein_2020/ss_grouped_vall_all.h5)
- This file will be used for some steps in cao's protocol to make better output.

## 11. Set the proper PATH to all binaries
#### (1) Move to $HOME and open .bashrc
    $cd ~
    $vi .bashrc
- It is strongly recommended to copy the .bashrc to backup directory for safe.

#### (2) Set the cursor in the empty line and change to "insert mode" by press "Shift + i" buttons and add the PATH.

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
        PATH=$PATH:/home/users/srgo/rosetta/rosetta_src_2021.16.61629_bundle/main/source/bin:/home/users/srgo/rosetta/rifdock/build/apps/rosetta:/home/users/srgo/rosetta/scaffolds:/home/users/srgo/rosetta/silent_tools
        export PATH

        LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
        export LD_LIBRARY_PATH
        
        module load gcc/6.5.0


#### (3) If you finished the revision, press "ESC" button to exit from "insert mode" and press ":" button and type "wq" and press "Enter" button to save the changes and exit from the file.
        :wq
        (Enter)

#### (4) Refresh the PATH by "source .bashrc" command and check whether the PATH is sucessfully added.
    $ source .bashrc
    $ echo $PATH
    
## 12. Try rifdock_tutorial
#### I'm still working to follow the cao's protocol. 
#### I will update the entire process after I finished to follow the protocol.

## Glossaries
### Basic programming concepts
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
- CMake: The program which generates build files(such as Makefile) that build a specific project. It is not the building program itself, but it is a program to generate build files. CMakeList.txt --(CMake)--> Makefile --(make)--> Excutables 
In CMakeList.txt, there are two kinds of information. One is the minimum required version of CMake and another is the information about the project. For CMake has been changed from previous old ones(especially ver. 2.x), you need to identify whether the CMake which is installed in your computer is compatible to do its proper function.
- make: File Maganement Utility which execute compile based on the dependencies described in "Makefile"
- makefile: Setting file for "make" program, which could simplify the iterative compile process. It defines macro, targets, rules, commands, and dependancies to compile source files with make command.
- C++11: New standard of C++. ISO assigned it at 12th Agust 2011. It made from gcc 4.3 to 4.8. Multi-threading programming API was standardized in this version. So, multi-thread programming could be performed without limitation of OS(Windows, Linux, IOS, and Android).\
- API: 
- OpenMP:
- OpenMPI:

### Basic concepts to install program in linux
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
