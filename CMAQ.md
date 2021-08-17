# CMAQ v5.3.2 Porting Guide(CentOS 7.6)

## Introduction
The Community Multiscale Air Quality (CMAQ) modeling system is at the heart of the third-generation air quality modeling system (Models-3) developed by the U.S. Environmental Protections Agency (USEPA). It is a three-dimensional Euler grid-based atmospheric chemistry and transmission simulation system to provide sound estimates of ozone, particulates, toxics, and acid deposition. The CMAQ model comprehensively represents air quality problems at different spatial scales and is widely used to address environmental issues and to conduct atmospheric research.

For more information, visit the CMAQ official website.

Programming language: Fortran

Brief description: a three-dimensional Euler grid-based atmospheric chemistry and transmission simulation system

## Environment Requirements

### Software Requirements
| Item  | Version  |  Download Address |
| ------------ | ------------ | ------------ |
|    HDF5      |    1.10.1    | https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.1/src/     |
|    PNETCDF   |    1.9.0     | https://parallel-netcdf.github.io/wiki/Download.html            |
|    NETCDF-C  |    4.7.0     | https://github.com/Unidata/netcdf-c/releases/tag/v4.7.0           |
|    NETCDF-F  |    4.5.5     | https://github.com/Unidata/netcdf-fortran/releases/tag/v4.4.5           |
|    IOAPI     |    3.2       | https://codeload.github.com/cjcoats/ioapi-3.2/tar.gz/2020111
|    CMAQ      |    5.3.1     | https://codeload.github.com/USEPA/CMAQ/tar.gz/CMAQv5.3.1_19Dec2019             |

### OS Requirements

| Item    | Version    | How to Obtain                    |
| ------- | ---------- | -------------------------------- |
| Cent OS | 7.6        | https://www.centos.org/download/ |
| Kernel  | 4.14.0-115 | Included in the OS image         |

## Configuring the Compilation Environment

### Installing dependencies


yum install time -y
yum install curl* -y
yum install csh -y
yum install zlib* -y


### Installing GNU 9.3


yum install -y centos-release-scl
yum install -y devtoolset-9-gcc
yum install -y devtoolset-9-gcc-c++
yum install -y devtoolset-9-binutils
scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile


### Installing Open MPI

1. Install the system dependency package:


yum install libxml2* systemd-devel.aarch64 numa* -y


2. Install Open MPI:


wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz
tar -zxvf openmpi-4.0.3.tar.gz
cd openmpi-4.0.3
./configure --prefix=/path/to/OPENMPI --enable-pretty-print-stacktrace --enable-orterun-prefix-by-default  --with-cma --enable-mpi1-compatibility
make -j 16
make install


3. Configure environment variables:


export PATH=/path/to/OPENMPI/bin:$PATH
export LD_LIBRARY_PATH=/path/to/OPENMPI/lib:$LD_LIBRARY_PATH


### Installing HDF5
1. Run the following commands to install HDF5:


    wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.1/src/hdf5-1.10.1.tar.gz
    tar -zxvf hdf5-1.10.6.tar.gz
    cd hdf5-1.10.1
    mkdir -p /path/to/HDF5
    ./configure --prefix=/path/to/HDF5 --build=aarch64-unknown-linux-gnu --enable-fortran --enable-static=yes --enable-parallel --enable-shared CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    make -j 16
    make install
2. Configure environment variables:


    export PATH=/path/to/hdf5/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/hdf5/lib:$LD_LIBRARY_PATH


### Installing PNETCDF
1. Run the following commands to install PNETCDF:


    wget https://parallel-netcdf.github.io/Release/pnetcdf-1.9.0.tar.gz
    tar -zxvf pnetcdf-1.9.0.tar.gz
    cd pnetcdf-1.9.0
    mkdir -p /path/to/pnetcdf

    ./configure --prefix=/path/to/PNETCDF --build=aarch64-unknown-linux-gnu CFLAGS="-fPIC -DPIC" CXXFLAGS="-fPIC -DPIC" FCFLAGS="-fPIC" FFLAGS="-fPIC" CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    make -j 16
    make install
2. Configure environment variables:


    export PATH=/path/to/pnetcdf/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/pnetcdf/lib:$LD_LIBRARY_PATH
### Installing NETCDF-C
1. Run the following commands to install NETCDF-C:


    wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.7.0.tar.gz
    tar -zxvf netcdf-c-4.7.0.tar.gz
    cd netcdf-c-4.7.0
    mkdir -p /path/to/netcdf
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --enable-netcdf-4 --enable-dap --with-pic --disable-doxygen --enable-static --enable-pnetcdf --enable-largefile CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/PNETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/PNETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -I/path/to/HDF5/include -I/path/to/PNETCDF/include"
    make -j 16
    make install
2. Configure environment variables:


    export PATH=/path/to/netcdf/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/netcdf/lib:$LD_LIBRARY_PATH
### Installing NETCDF-FORTRAN
1. Run the following commands to install NETCDF-FORTRAN:


    wget https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.4.5.tar.gz
    tar -zxvf netcdf-fortran-4.4.5.tar.gz
    cd netcdf-fortran-4.4.5
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --with-pic --disable-doxygen --enable-largefile --enable-static CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/NETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/NETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" CXXFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" FCFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include"
    make -j 16
    make install


## Compiling and installing CMAQ
1. Run the following command to obtain the source code package:


    wget https://codeload.github.com/cjcoats/ioapi-3.2/tar.gz/2020111
    wget https://codeload.github.com/USEPA/CMAQ/tar.gz/CMAQv5.3.1_19Dec2019
2. Run the following command to go to the CMAQ directory:

    cd /path/to/CMAQ
3. Run the following commands to decompress the package and rename ioapi:

    tar -xvf ioapi-3.2-2020111.tar.gz
    mv ioapi-3.2-2020111 ioapi-3.2
4. Run the following commands to copy the configuration file:

   cd ioapi-3.2
   cp ioapi/Makeinclude.Linux2_ia64gfort ioapi/Makeinclude.Linux4_aarch64 
5. Edit the configuration file.
   
   a. Run the following command to modify the Makeinclude.Linux4_aarch64 configuration file:
   
   vim ioapi/Makeinclude.Linux4_aarch64

   b. Press i to enter the editing mode.

   Modify the compiler options:
     CC   = mpicc
     CXX  = mpicxx
     FC   = mpifort
   
   Comment out the line corresponding to the FSFLAGS keyword, for example:

     #FSFLAGS = -save
   
   c. Press Esc, enter :wq!, and press Enter to save the file and exit.

6. Run the following commands to copy Makefile and configure HOME:

   cp ioapi/Makefile.nocpl ioapi/Makefile
   export HOME=/path/to/CMAQ
7. Run the following command to copy the configuration file:

   cp m3tools/Makefile.nocpl m3tools/Makefile

8. Edit the configuration file.

   a. Run the following command to modify the Makefile configuration file:

   vim m3tools/Makefile

   b. Press i to enter the editing mode and modify the script as follows:

   LIBS = -L${OBJDIR} -lioapi -L/path/to/NETCDF/lib -lnetcdff -lnetcdf -L/path/to/HDF5/lib -lhdf5_hl -lhdf5 -lz $(OMPLIBS) $(ARCHLIB) $(ARCHLIBS)

   c. Press Esc, enter :wq!, and press Enter to save the file and exit.

9. Run the following command to copy the configuration file:
    
   cp Makefile.template Makefile

10. Edit the configuration file.
    
    a. Run the following command to edit the configuration file:
    vim Makefile

    b. Press i to enter the editing mode.
    Modify the following content and delete the comment tag:

    BIN        = Linux4_aarch64
    BASEDIR    = ${PWD}
    INSTALL    = ${HOME}
    LIBINST    = $(INSTALL)/$(BIN)
    BININST    = $(INSTALL)/$(BIN)
    CPLMODE    = nocpl
    IOAPIDEFS  = "-DIOAPI_NCF4"

    Modify the NCFLIBS item:
    NCFLIBS    = -L/path/to/NETCDF/lib -lnetcdff -lnetcdf -L/path/to/HDF5/lib -lhdf5_hl -lhdf5 -lz

    c. Press Esc, enter :wq!, and press Enter to save the file and exit.

11. Run the following command to compile ioapi:
    
    make BIN=Linux4_aarch64

12. Modify the STATE3.EXT file.
    
    a. Run the following command to modify the STATE3.EXT file:
    vim ioapi/STATE3.EXT

    b. Press i to enter the editing mode. Delete & at the end of some lines in the STATE3.EXT file.

    c. Press Esc, enter :wq!, and press Enter to save the file and exit.

13. Run the following commands to decompress the package and go to the directory:
    
    tar -xvf CMAQ-CMAQv5.3.1_19Dec2019.tar.gz
    cd CMAQ-CMAQv5.3.1_19Dec2019

14. Edit the configuration file.

    a. Run the following command to edit the configuration file:
    vim bldit_project.csh

    b. Press i to enter the editing mode and modify the script as follows:
    set CMAQ_HOME = /path/to/CMAQ/CMAQ_Project

    c. Press Esc, enter :wq!, and press Enter to save the file and exit.

15. Run the following command to create the files required for initialization:
    
    ./bldit_project.csh

16. Run the following command to switch to the working directory:
    
    cd ../CMAQ_Project/

17. Edit the configuration file.
    
    a. Run the following command to edit the configuration file:
       vim config_cmaq.csh

    b. Press i to enter the editing mode.
       In the case gcc area, modify the paths to dependencies, as shown in the following figure:

       setenv IOAPI_MOD_DIR   /path/to/CMAQ/ioapi-3.2/Linux4_aarch64/
       setenv IOAPI_INCL_DIR  /path/to/CMAQ/ioapi-3.2/ioapi/
       setenv IOAPI_LIB_DIR   /path/to/CMAQ/ioapi-3.2/Linux4_aarch64/
       setenv NETCDF_LIB_DIR  /path/to/NETCDF/lib/
       setenv NETCDF_INCL_DIR /path/to/NETCDF/include/
       setenv MPI_LIB_DIR     /path/to/OPENMPI/

       Modify the compiler parameters. For example:

       setenv myCC mpicc
       setenv myLINK_FLAG  "-fopenmp"
       setenv mpi_lib "-lmpi"

       Add the openmp attribute to the netcdf_lib variable. For example:
    
       setenv netcdf_lib "-lnetcdf -lnetcdff -lgomp"  #> -lnetcdff -lnetcdf for netCDF v4.2.0 and later

    c. Press Esc, enter :wq!, and press Enter to save the file and exit.

18. Run the following command to connect to various dependent libraries:
    
    ./config_cmaq.csh gcc 9.3.0

19. Run the following commands in sequence to go to the compilation directory and compile the main program:
    
    cd CCTM/scripts/
    ./bldit_cctm.csh gcc 9.3.0

After the compilation is complete, the executable program CCTM_v531.exe is generated in the BLD_CCTM_v531_gcc9.3.0 directory.

