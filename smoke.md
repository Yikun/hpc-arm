# SMOKE 4.8.1 Porting Guide(CentOS 7.6)
## Introduction
The Sparse Matrix Operator Kernel Emissions (SMOKE) modeling system is primarily an emissions processing system designed to create gridded, speciated, hourly emissions for input into a variety of air quality models, such as CMAQ, REMSAD, CAMX, and UAM. SMOKE supports area, biogenic, mobile (both onroad and nonroad), and point source emissions processing for criteria, particulate, and toxic pollutants. For biogenic emissions modeling, SMOKE uses the Biogenic Emission Inventory System, version 2.5 (BEIS2) and version 3.09 and 3.14 (BEIS3). SMOKE is also integrated with the onroad emissions model MOBILE6 and MOVES.

For more information about the SMOKE, visit the official SMOKE website.

Programming language: Fortran 90

Brief description: source emission tool for air modeling using sparse matrix

Open-source protocol: LGPL-3.0

##### Recommended Version
The recommended version is SMOKE V4.8.1

## Environment Requirements

### Software Requirements
| Item  | Version  |  Download Address |
| ------------ | ------------ | ------------ |
|  OPENMPI  |  4.0.3  | https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz  |
|  HDF5     |  1.10.1 | https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.1/src/hdf5-1.10.1.tar.gz  |
|  PNETCDF  |  1.8.0  |  https://parallel-netcdf.github.io/Release/parallel-netcdf-1.8.0.tar.gz |
|  NETCDF-C | 4.4.1.1 |  https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.4.1.1.tar.gz |
|  NETCDF-F | 4.4.1   | https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.4.1.tar.gz  |
|  IOAPI    | 3.2     | https://github.com/cjcoats/ioapi-3.2  |
|  SMOKE    | v481    | https://github.com/CEMPD/SMOKE/releases  |

### OS Requirements
| Item  | Version  | How to Obtain  |
| ------------ | ------------ | ------------ |
|  CentOS | 7.6  |  https://www.centos.org/download/ |
| Kernel  | 4.14.0-115  |  Included in the OS image. |

## Configuring the Compilation Environment

### Installing dependencies


```shell
yum install time -y
yum install curl* -y
yum install wget -y
yum install csh -y
yum install zlib* -y
```

### Installing GNU 9.3

```shell
yum install -y centos-release-scl
yum install -y devtoolset-9-gcc
yum install -y devtoolset-9-gcc-c++
yum install -y devtoolset-9-binutils
scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
```

### Installing Open MPI
1. Run the following command to install the system dependency package:
```

    yum install libxml2* systemd-devel.aarch64 numa* -y
```
2. Run the following commands to install Open MPI:

```
    wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz
    tar -zxvf openmpi-4.0.3.tar.gz
    cd openmpi-4.0.3
    ./configure --prefix=/path/to/OPENMPI --enable-pretty-print-stacktrace --enable-orterun-prefix-by-default  --with-cma --enable-mpi1-compatibility --enable-mpi-fortran=yes
    make -j $(nproc) all
    make install
```
3. Configure environment variables:

```
    export PATH=/path/to/OPENMPI/bin:$PATH
    export LD_LIBRARY_PATH=/path/to/OPENMPI/lib:$LD_LIBRARY_PATH
```
### Installing HDF5
1. Run the following commands to install HDF5:
```

    wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.1/src/hdf5-1.10.1.tar.gz
    
    tar -zxvf hdf5-1.10.1.tar.gz
    
    cd hdf5-1.10.1
    
    ./configure --prefix=/path/to/HDF5 --build=aarch64-unknown-linux-gnu --enable-fortran --enable-static=yes --enable-parallel --enable-shared CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    
    make -j $(nproc)
    
    make install
```
2. Configure environment variables:
```

    export PATH=/path/to/HDF5/bin:$PATH
    
    export LD_LIBRARY_PATH=/path/to/HDF5/lib:$LD_LIBRARY_PATH
```
### Installing PNETCDF
1. Run the following commands to install PNETCDF:
```

    wget https://parallel-netcdf.github.io/Release/parallel-netcdf-1.8.0.tar.gz
    
    tar -zxvf parallel-netcdf-1.8.0.tar.gz
    
    cd parallel-netcdf-1.8.0
    
    ./configure --prefix=/path/to/PNETCDF --build=aarch64-unknown-linux-gnu CFLAGS="-fPIC -DPIC" CXXFLAGS="-fPIC -DPIC" FCFLAGS="-fPIC" FFLAGS="-fPIC" CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    
    make -j $(nproc)
    
    make install
```
2. Configure environment variables:

```
    export PATH=/path/to/PNETCDF/bin:$PATH
    
    export LD_LIBRARY_PATH=/path/to/PNETCDF/lib:$LD_LIBRARY_PATH
```
### Installing NETCDF-C
1. Run the following commands to install NETCDF-C:

```
    wget -O netcdf-c-4.4.1.1.tar.gz https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.4.1.1.tar.gz
    
    tar -zxvf netcdf-c-4.4.1.1.tar.gz
    
    cd netcdf-c-4.4.1.1
    
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --enable-netcdf-4 --enable-dap --with-pic --disable-doxygen --enable-static --enable-pnetcdf --enable-largefile CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/PNETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/PNETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -I/path/to/HDF5/include -I/path/to/PNETCDF/include"
    
    make -j $(nproc)
    
    make install
```
2. Configure environment variables:
```

    export PATH=/path/to/NETCDF/bin:$PATH
    
    export LD_LIBRARY_PATH=/path/to/NETCDF/lib:$LD_LIBRARY_PATH
```
### Installing NETCDF-FORTRAN
1. Run the following commands to install NETCDF-FORTRAN:

```
    wget -O netcdf-fortran-4.4.1.tar.gz https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.4.1.tar.gz
    
    tar -zxvf netcdf-fortran-4.4.1.tar.gz
    
    cd netcdf-fortran-4.4.1
    
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --with-pic --disable-doxygen --enable-largefile --enable-static CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/NETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/NETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" CXXFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" FCFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include"
    
    make -j $(nproc)
    
    make install
```
### Installing IOAPI
1. Run the following command to install SMOKE in csh:
```
   chsh -s /bin/csh
```
2. Restart the system for the installation to take effect.
```
   reboot
```
3. Set environment variables.
```
   echo "setenv SMK_HOME /path/to/SMOKE" >> /root/.tcsh

   echo "setenv BIN Linux2_aarch64gfort" >> /root/.tcsh

   echo "setenv LD_LIBRARY_PATH /opt/rh/devtoolset-9/root/lib64:/path/to/OPENMPI/lib" >> /root/.tcsh

   echo setenv PATH /path/to/OPENMPI/bin:/opt/rh/devtoolset-9/root/bin:$PATH>> /root/.tcsh

   echo setenv LD_LIBRARY_PATH /path/to/NETCDF/lib:$LD_LIBRARY_PATH >> /root/.tcsh
```
4. Run the following command to make the environment variables take effect:
```

   source /root/.tcsh
```
5. Copy the installation packages to the installation paths:
```
   cp smoke_v481.Linux2_x86_64ifort.tar.gz /path/to/SMOKE

   cp smoke_v481.nctox.data.tar.gz /path/to/SMOKE

   cp smoke_install_v481.csh /path/to/SMOKE

   cp ioapi-3.2.tar.gz /path/to/SMOKE

   cd /path/to/SMOKE

   source smoke_install_v481.csh
```
6. Create the directory:
```
   mkdir -p $SMK_HOME/subsys/ioapi

   mkdir -p $SMK_HOME/subsys/ioapi/$BIN
```
7. Decompress the ioapi-3.2.tar.gz installation package:
```
   cp ioapi-3.2.tar.gz ./subsys/ioapi
  
   cd ./subsys/ioapi
  
   tar -xvf ioapi-3.2.tar.gz
```
8. Create and Modify the Makefile file
```
   cd $SMK_HOME/subsys/ioapi/ioapi

   cp Makefile.nocpl Makefile
```
   Edit Makefile and modify the following parameters:
   
```
   BASEDIR = ${SMK_HOME}/subsys/ioapi
   INSTDIR = ${BASEDIR}/${BIN}
```
9. Modify the Makeinclude.Linux2_aarch64gfort file
```
   cp Makeinclude.Linux2_x86_64gfort Makeinclude.$BIN
```
   Edit Makeinclude and modify the following parameters:

```
   MFLAGS = -ffast-math -funroll-loops -march=armv8-a
```
10. Run the following command to perform compilation:
```
     make
```
11. Run the following commands to set soft links:
```
    cd ../$BIN

    ln -sf /path/to/NETCDF/lib/libnetcdf.so ./

    ln -sf /path/to/NETCDF/lib/libnetcdff.so ./
```
12. Run the following command to switch to the m3tools directory:
```
    cd ../m3tools
```
13. Modify the Makefile file.
```
    cp Makefile.nocpl Makefile
```
    Edit Makeinclude and modify the following parameters:
```
    BASEDIR = ${SMK_HOME}/subsys/ioapi

    INSTDIR = ${BASEDIR}/${BIN}
```
14. Run the following command to perform compilation:
```
    make
```
## Compiling and Installing SMOKE
1. Run the following command:
```
   cd $SMK_HOME/subsys/smoke/assigns
```
2. Modify the configuration file.

   Edit ASSIGNS.nctox.cmaq.cb05_soa.us12-nc and modify line 25 as follows:
```
   setenv BIN   Linux2_aarch64gfort
```
3. Modify the Makeinclude file.
```
   cd $SMK_HOME/subsys/smoke/src

   vi Makeinclude
```
```
   INSTDIR = ${OBJDIR}/${BIN}

   #EFLAG = -extend-source 132 -zero 

   EFLAG = -ffixed-line-length-132  -fno-backslash
```
4. Modify the smkinven/rdinvsrcs.f file.

   vi smkinven/rdinvsrcs.f
   
   Change the type of the GETPID function from INTEGER*4 to INTRINSIC.

5. Modify the smkinven/rdemspd.f file.

   vi smkinven/rdemspd.f

   Add a capital letter "C" on line 304 to comment on code 304.
```
   304  C    LFIP = ''
```
6. Create the directory.
```
   mkdir ${SMK_HOME}/subsys/smoke/${BIN}
```
7. Run the following commands to complete compilation:
```
   source /path/to/SMOKE/subsys/smoke/assigns/ASSIGNS.nctox.cmaq.cb05_soa.us12-nc

   make
```
## Running and Verifying SMOKE
1. Run the following command to switch to test directory::
```
   cd $SMK_HOME/subsys/smoke/script/run/
```
2. Run the following command to test the execution time of each script:
```
   time ./Script name
   Example: time ./smk_point_nctox.csh
```
