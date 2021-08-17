# NEMO v4.0.6 Porting Guide(Cent OS 7.6)

## Introduction



## Environment Requirements

### Software Requirements

| Item     | Version | Download Address                                             |
| -------- | ------- | ------------------------------------------------------------ |
| OPENMPI  | 4.0.3   | https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz  |
| HDF5     | 1.10.6  | https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/ |
| PNETCDF  | 1.12.1  | https://parallel-netcdf.github.io/wiki/Download.html         |
| NETCDF-C | 4.7.3   | https://github.com/Unidata/netcdf-c/releases/tag/v4.7.3      |
| NETCDF-F | 4.5.2   | https://github.com/Unidata/netcdf-fortran/releases/tag/v4.5.2 |
| XIOS     | 2.5     | https://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/branchs/xios-2.5 |
| NEMO     | 4.0.2   | https://forge.ipsl.jussieu.fr/nemo/svn/NEMO/releases/r4.0/r4.0.6 |

### OS Requirements

| Item   | Version    | How to Obtain                    |
| ------ | ---------- | -------------------------------- |
| CentOS | 7.6        | https://www.centos.org/download/ |
| Kernel | 4.14.0-115 | Included in the OS image.        |

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
### Installing XIOS

1. Get the source code:

```shell
svn co https://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/branchs/xios-2.5
```

2. Create the arch files:

`vim arch/arch-AARCH64_LINUX.env`

```shell
#PATH to install HDF5
export HDF5_INC_DIR="/path/to/HDF5/include"
export HDF5_LIB_DIR="/path/to/HDF5/lib"
#PATH to install NETCDF
export NETCDF_INC_DIR="/path/to/NETCDF/include"
export NETCDF_LIB_DIR="/path/to/NETCDF/lib"
```

`vim arch/arch-AARCH64_LINUX.fcm`

```shell
################################################################################
###################        Projet xios - xmlioserver       #####################
################################################################################

%CCOMPILER      mpicc
%FCOMPILER      mpif90
%LINKER         mpif90  

%BASE_CFLAGS    -ansi -w
%PROD_CFLAGS    -O3 -DBOOST_DISABLE_ASSERTS
%DEV_CFLAGS     -g -O2 
%DEBUG_CFLAGS   -g 

%BASE_FFLAGS    -D__NONE__ 
%PROD_FFLAGS    -O3
%DEV_FFLAGS     -g -O2
%DEBUG_FFLAGS   -g 

%BASE_INC       -D__NONE__
%BASE_LD        -lstdc++

%CPP            cpp
%FPP            cpp -P
%MAKE           gmake
```

`vim arch/arch-AARCH64_LINUX.path`

```shell
NETCDF_INCDIR="-I /path/to/NETCDF/include"
NETCDF_LIBDIR="-L /path/to/NETCDF/lib"
NETCDF_LIB="-lnetcdff -lnetcdf"

MPI_INCDIR="-I /path/to/OpenMPI/include"
MPI_LIBDIR="-L /path/to/OpenMPI/lib"
MPI_LIB="-lmpi"

HDF5_INCDIR="-I /path/to/HDF5/include"
HDF5_LIBDIR="-L /path/to/HDF5/lib"
HDF5_LIB="-lhdf5_hl -lhdf5  -lz"
```

3. Run the build script:

```shell
./make_xios --job 32 --full --arch AARCH64_HCC_LINUX -dev
```



## Compiling and Installing NEMO

Run the following command to obtain the source code package:


    svn co https://forge.ipsl.jussieu.fr/nemo/svn/NEMO/releases/r4.0/r4.0.6
    cd r4.0.6

Edit the arch files:

`vim arch/arch-aarch64_linux_gnu.fcm`


```shell
%NCDF_HOME           /path/to/NETCDF
%HDF5_HOME           /path/to/HDF5/
%XIOS_HOME           /path/to/xios-2.5

%NCDF_INC            -I%NCDF_HOME/include -I%HDF5_HOME/include
%NCDF_LIB            -L%HDF5_HOME/lib -L%NCDF_HOME/lib -lnetcdff -lnetcdf
%XIOS_INC            -I%XIOS_HOME/inc
%XIOS_LIB            -L%XIOS_HOME/lib -lxios

%CPP	             cpp -Dkey_nosignedzero
%FC	             mpif90 -c -cpp
%FCFLAGS             -mcpu=native -fdefault-real-8 -fdefault-double-8 -Ofast -funroll-all-loops -fcray-pointer -ffree-line-length-none -g -ffree-form
%FFLAGS              %FCFLAGS
%LD                  mpif90
%LDFLAGS             -lstdc++
%FPPFLAGS            -P -C -traditional
%AR                  ar
%ARFLAGS             rs
%MK                  gmake
%USER_INC            %XIOS_INC %NCDF_INC
%USER_LIB            %XIOS_LIB %NCDF_LIB -lm

%CC                  cc
%CFLAGS              -O0
```

Change to the `config` directory and run the build script:


```shell
cd config
./makenemo -m aarch64_linux_gnu -r 'target_configure' -n 'your_configure'
```

Check whether the installation is successful:


    ls -l your_configure/EXP00/nemo

If the executable  file is generated correctly, the installation is completed successful.
