# ROMS 3.6 Porting Guide(CentOS 7.6)
## Introduction
Regional Ocean Modeling System is a three-dimensional regional ocean model, jointly developed by the Institute of Marine and Coastal Sciences of Rutgers University and the University of California, Los Angeles. It is widely used to simulate the hydrodynamic and water environment of the ocean and estuarine region.

For more information about the ROMS, visit the official ROMS website.

Programming language: Fortran

Brief description: three-dimensional regional ocean model

Open-source protocol: MIT/X License

##### Recommended Version
The recommended version is ROMS V3.6

## Environment Requirements

### Software Requirements
| Item         | Version  |  Download Address                                                          |
| ------------ | -------- | -------------------------------------------------------------------------- |
|  OPENMPI     |  4.0.3   |  https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz  |
|  HDF5        |  1.10.6  |  https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/ |
|  PNETCDF     |  1.12.1  |  https://parallel-netcdf.github.io/wiki/Download.html                      |
|  NETCDF-C    |  4.7.3   |  https://github.com/Unidata/netcdf-c/releases/tag/v4.7.3                   |
|  NETCDF-F    |  4.5.2   |  https://github.com/Unidata/netcdf-fortran/releases/tag/v4.5.2             |
|  ROMS        |  3.6     |  https://www.myroms.org/svn/src/trunk                                      |

### OS Requirements
| Item    | Version     | How to Obtain                     |
| ------- | ----------- | --------------------------------- |
| CentOS  | 7.6         |  https://www.centos.org/download/ |
| Kernel  | 4.14.0-115  |  Included in the OS image.        |

## Configuring the Compilation Environment

### Installing dependencies

```shell
yum install time -y
yum install curl* -y
yum install libcurl-devel -y
yum install wget -y
yum install csh -y
yum install zlib* -y
```

### Installing GNU 9.3


    yum install -y centos-release-scl
    yum install -y devtoolset-9-gcc
    yum install -y devtoolset-9-gcc-c++
    yum install -y devtoolset-9-binutils
    scl enable devtoolset-9 bash
    echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
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
    wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/hdf5-1.10.6.tar.gz
    tar -zxvf hdf5-1.10.6.tar.gz
    cd hdf5-1.10.6
    mkdir -p /path/to/HDF5
    ./configure --prefix=/path/to/HDF5 --build=aarch64-unknown-linux-gnu --enable-fortran --enable-static=yes --enable-parallel --enable-shared CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    make -j 16
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
    wget https://parallel-netcdf.github.io/Release/pnetcdf-1.12.1.tar.gz
    tar -zxvf pnetcdf-1.12.1.tar.gz
    cd pnetcdf-1.12.1
    mkdir -p /path/to/PNETCDF

    ./configure --prefix=/path/to/PNETCDF --build=aarch64-unknown-linux-gnu CFLAGS="-fPIC -DPIC" CXXFLAGS="-fPIC -DPIC" FCFLAGS="-fPIC" FFLAGS="-fPIC" CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
    make -j 16
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
    wget -O netcdf-c-4.7.3.tar.gz https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.7.3.tar.gz
    tar -zxvf netcdf-c-4.7.3.tar.gz
    cd netcdf-c-4.7.3
    mkdir -p /path/to/NETCDF
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --enable-netcdf-4 --enable-dap --with-pic --disable-doxygen --enable-static --enable-pnetcdf --enable-largefile CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/PNETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/PNETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -I/path/to/HDF5/include -I/path/to/PNETCDF/include"
    make -j 16
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

    wget -O netcdf-fortran-4.5.2.tar.gz https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.5.2.tar.gz
    tar -zxvf netcdf-fortran-4.5.2.tar.gz
    cd netcdf-fortran-4.5.2
    ./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --with-pic --disable-doxygen --enable-largefile --enable-static CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/NETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/NETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" CXXFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" FCFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include"
    make -j 16
    make install
```
## Compiling and Installing ROMS
1. Run the following command to obtain the source code package:
```
    cd /path/to/ROMS
    mkdir ROMS_3.6
    svn checkout --username your_username --password your_password https://www.myroms.org/svn/src/trunk ROMS_3.6
```
2. Create folders for compiling and running the ROMS.:
```
    cd /path/to/ROMS
    mkdir -p ROMSProjects/upwelling
    cd ROMSProjects/upwelling
```
3. Edit the build.bash file:
```
    cp /path/to/ROMS/ROMS_3.6/ROMS/Bin/build.bash ./
    vim build.bash
```
4. Modify the following information in the build.bash file:
```
    export ROMS_APPLICATION=UPWELLING

    export MY_ROOT_DIR=/path/to/ROMS
    export MY_PROJECT_DIR=${MY_ROOT_DIR}/ROMSProjects/upwelling

    export MY_ROMS_SRC=/path/to/ROMS/ROMS_3.6

    export USE_MPI=on
    export USE_MPIF90=on
    export which_MPI=openmpi

    export FORT=gfortran

    export USE_LARGE=on
    export USE_NETCDF4=on
    export USE_PARALLEL_IO=on

    export PATH=/path/to/OPENMPI/bin:$PATH   ## Modifying the gfortran-openmpi Branch ##

    export NF_CONFIG=/path/to/NETCDF/bin/nf-config  ## Modifying the gfortran-openmpi Branch ##
    export NETCDF_INCDIR=/path/to/NETCDF/include  ## Modifying the gfortran-openmpi Branch ##
    export NETCDF_LIBDIR=/path/to/NETCDF/lib  ## Modifying the gfortran-openmpi Branch ##
```
5. Copy the input file and config file:
```
    cp /path/to/ROMS/ROMS_3.6/ROMS/External/ocean_upwelling.in ./
    cp /path/to/ROMS/ROMS_3.6/ROMS/External/varinfo.dat ./
    cp /path/to/ROMS/ROMS_3.6/ROMS/Include/upwelling.h ./
```
6. Edit the ocean_upwelling.in file:
```
    vim ocean_upwelling.in
```
7. Modify the following information in the ocean_upwelling.in file:
```
    VARNAME = /path/to/ROMS/ROMSProjects/upwelling/varinfo.dat

    NtileI == 6
    NtileJ == 16
   tips:6*16=nproc(This number indicates how many processes you want to run ROMS) 
```
8. Run the following command to install ROMS:
```
    ./build.bash -j
```
9. Check whether the installation is successful:

```
    ls main
```
If the following information is displayed (the oceanM file is generated), the installation is successful.
```
    oceanM
```
