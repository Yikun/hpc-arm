# fvcom 4.3 Porting Guide(CentOS 7.6)
## Introduction
A prognostic, unstructured-grid, finite-volume, free-surface, 3-D primitive equation coastal ocean circulation model developed by UMASSD-WHOI joint efforts.

For more information about the fvcom, visit the official fvcom website.

Programming language: C/Fortran

Open-source protocol: GPL 3.0

##### Recommended Version
The recommended version is fvcom 4.3

## Environment Requirements

### Software Requirements
| Item      | Version |  Download Address |
| --------- | ------- | -------------------------------------------------------------------------- |
|  OPENMPI  |  4.0.3  |  https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz  |
|  HDF5     |  1.10.6 |  https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/ |
|  PNETCDF  |  1.12.1 |  https://parallel-netcdf.github.io/wiki/Download.html                      |
|  NETCDF-C |  4.7.3  |  https://github.com/Unidata/netcdf-c/releases/tag/v4.7.3                   |
|  NETCDF-F |  4.5.2  |  https://github.com/Unidata/netcdf-fortran/releases/tag/v4.5.2             |
|  WRF      |  4.3    |  http://fvcom.smast.umassd.edu/releases/fvcom-4.3.tar.gz                   |

### OS Requirements
| Item    | Version     | How to Obtain                     |
| ------- | ----------- | --------------------------------- |
|  CentOS | 7.6         |  https://www.centos.org/download/ |
|  Kernel | 4.14.0-115  |  Included in the OS image.        |

## Configuring the Compilation Environment

### Installing dependencies

```shell
yum install time -y
yum install curl* -y
yum install libcurl-devel -y
yum install wget -y
yum install csh -y
yum install zlib* -y
yum install perl -y
yum install make -y
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

```shell
yum install libxml2* systemd-devel.aarch64 numa* -y
```

2. Run the following commands to install Open MPI:

```shell
wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz
tar -zxvf openmpi-4.0.3.tar.gz
cd openmpi-4.0.3
./configure --prefix=/path/to/OPENMPI --enable-pretty-print-stacktrace --enable-orterun-prefix-by-default  --with-cma --enable-mpi1-compatibility --enable-mpi-fortran=yes
make -j $(nproc) all
make install
```

3. Configure environment variables:

```shell
export PATH=/path/to/OPENMPI/bin:$PATH
export LD_LIBRARY_PATH=/path/to/OPENMPI/lib:$LD_LIBRARY_PATH
```

### Installing HDF5
1. Run the following commands to install HDF5:

```shell
wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/hdf5-1.10.6.tar.gz
tar -zxvf hdf5-1.10.6.tar.gz
cd hdf5-1.10.6
mkdir -p /path/to/HDF5
./configure --prefix=/path/to/HDF5 --build=aarch64-unknown-linux-gnu --enable-fortran --enable-static=yes --enable-parallel --enable-shared CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
make -j $(nproc)
make install
```

2. Configure environment variables:

```shell
export PATH=/path/to/HDF5/bin:$PATH
export LD_LIBRARY_PATH=/path/to/HDF5/lib:$LD_LIBRARY_PATH
```

### Installing PNETCDF
1. Run the following commands to install PNETCDF:

```shell
wget https://parallel-netcdf.github.io/Release/pnetcdf-1.12.1.tar.gz
tar -zxvf pnetcdf-1.12.1.tar.gz
cd pnetcdf-1.12.1
mkdir -p /path/to/PNETCDF

./configure --prefix=/path/to/PNETCDF --build=aarch64-unknown-linux-gnu CFLAGS="-fPIC -DPIC" CXXFLAGS="-fPIC -DPIC" FCFLAGS="-fPIC" FFLAGS="-fPIC" CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort
make -j $(nproc)
make install
```

2. Configure environment variables:

```shell
export PATH=/path/to/PNETCDF/bin:$PATH
export LD_LIBRARY_PATH=/path/to/PNETCDF/lib:$LD_LIBRARY_PATH
```

### Installing NETCDF-C
1. Run the following commands to install NETCDF-C:

```shell
wget -O netcdf-c-4.7.3.tar.gz https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.7.3.tar.gz
tar -zxvf netcdf-c-4.7.3.tar.gz
cd netcdf-c-4.7.3
mkdir -p /path/to/NETCDF
./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --enable-netcdf-4 --enable-dap --with-pic --disable-doxygen --enable-static --enable-pnetcdf --enable-largefile CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/PNETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/PNETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/PNETCDF/lib -I/path/to/HDF5/include -I/path/to/PNETCDF/include"
make -j $(nproc)
make install
```

2. Configure environment variables:

```shell
export PATH=/path/to/NETCDF/bin:$PATH
export LD_LIBRARY_PATH=/path/to/NETCDF/lib:$LD_LIBRARY_PATH
```

### Installing NETCDF-FORTRAN
1. Run the following commands to install NETCDF-FORTRAN:

```shell
wget -O netcdf-fortran-4.5.2.tar.gz https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.5.2.tar.gz
tar -zxvf netcdf-fortran-4.5.2.tar.gz
cd netcdf-fortran-4.5.2
./configure --prefix=/path/to/NETCDF --build=aarch64-unknown-linux-gnu --enable-shared --with-pic --disable-doxygen --enable-largefile --enable-static CC=mpicc CXX=mpicxx FC=mpifort F77=mpifort CPPFLAGS="-I/path/to/HDF5/include -I/path/to/NETCDF/include" LDFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -Wl,-rpath=/path/to/HDF5/lib -Wl,-rpath=/path/to/NETCDF/lib" CFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" CXXFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include" FCFLAGS="-L/path/to/HDF5/lib -L/path/to/NETCDF/lib -I/path/to/HDF5/include -I/path/to/NETCDF/include"
make -j $(nproc)
make install
```

## Compiling and Installing fvcom
1. Run the following command to obtain the source code package:

```shell
cd /path/to/FVCOM
wget http://fvcom.smast.umassd.edu/releases/fvcom-4.3.tar.gz
```

2. Decompress the WRF installation package:

```shell
tar -zxvf fvcom-4.3.tar.gz
```

3. Run the following command to switch to WRF source code directory:

```shell
cd fvcom-4.3
```

4. Run the following commands to configure related files:

```shell
cp Examples/Estuary/make.inc_example FVCOM_source/make.inc
ln -sf FVCOM_source/make.inc ./
```

5. Edit the make.inc file:

```shell
vim make.inc
```

Line 51:

```shell
TOPDIR = /path/to/FVCOM/FVCOM4.1/FVCOM_source
```

Line 79,80:

```shell
LIBDIR = -L$(INSTALLDIR)/lib -L../METIS_source/metis -L./libs/julian
INCDIR = -I$(INSTALLDIR)/include -I../METIS_source/metis -I./libs/julian
```

Line 97:

```shell
IOLIBS = -L/path/to/NETCDF/lib -L/path/to/HDF5/lib -lnetcdff -lnetcdf -lhdf5_hl -lhdf5 -lz -lcurl -lm
```

Line 989:

```shell
IOINCS = -I/path/to/NETCDF/include -I/path/to/HDF5/include
```

Line 532:

```
    #  Intel/MPI Compiler Definitions (SMAST)
    #--------------------------------------------------------------------------
    #         CPP      = /usr/bin/cpp
    #         COMPILER = -DIFORT
    #         CC       = mpicc
    #         CXX      = mpicxx
    #         CFLAGS   = -O3
    #         FC       = mpif90
    #         DEBFLGS  = -check all -traceback
    # Use 'OPT = -O0 -g'  for fast compile to test the make
    # Use 'OPT = -xP' for fast run on em64t (Hydra and Guppy)
    # Use 'OPT = -xN' for fast run on ia32 (Salmon and Minke)
    #         OPT      = -O0 -g
    #         OPT      = -axN -xN
    #         OPT      = -O3
    
    # Do not set static for use with visit!
    #         VISOPT   = -Wl,--export-dynamic
    #         LDFLAGS  =  $(VISITLIBPATH)
    #--------------------------------------------------------------------------
    #  gfortran defs
    #--------------------------------------------------------------------------
             CPP      = /usr/bin/cpp
             COMPILER = -DGFORTRAN
             CC       = mpicc
             CXX      = mpicxx
             FC       = mpif90
             DEBFLGS  =
             OPT      = -O3 -ffixed-line-length-none -ffree-form -ffree-line-length-none
             CLIB     =
```

6. Run the following command to compile the METIS:

```shell
cd ./METIS_source
tar -zxvf metis.tgz
wget https://www.math-linux.com/IMG/patch/metis-4.0.patch --no-check-certificate
cd metis
patch -p2 < ../metis-4.0.patch
mkdir -p /path/to/FVCOM/FVCOM4.3/FVCOM_source/libs/install/lib
mkdir -p /path/to/FVCOM/FVCOM4.3/FVCOM_source/libs/install/include
mkdir -p /path/to/FVCOM/FVCOM4.3/FVCOM_source/libs/install/bin
make -j $(nproc)
make install
```

7. Run the following command to compile the julian:

```shell
cd ../../FVCOM_source/libs
tar -zxvf julian.tgz
cd julian
make -j $(nproc)
make install
```

8. Run the following command to modify some source code:

```shell
cd /path/to/FVCOM/FVCOM4.1/FVCOM_source

vim mod_newinp.F
```

Line 343:
```
    write(*,N_Fmt('(A20,<size>F10.4)',SIZE))trim(argname)//': ',fval(1:SIZE)
```

Line 412:
```
    write(*,N_Fmt('(A20,<size>I10)',SIZE))trim(argname)//': ',ival(1:SIZE)
```

Line 485:
```
    write(*,N_Fmt('(A20,<size>L10)',SIZE))trim(argname)//': ',cval(1:SIZE)
```

Line 558:
```
    write(*,N_Fmt('(A20,<size>A10)',SIZE))trim(argname)//': ',sval(1:SIZE)
```

Line 50:
```
    contains
      Character( Len = 256 ) Function N_Fmt( c , n )
        Character( Len = * ) , Intent( IN ) :: c
        Integer , Intent( IN ) :: n
        integer :: i , j
        character( len = 16 ) :: cn
        i = index( c , '<' )
        j = index( c , '>' )
        write( cn , '(g0)' ) n
        N_Fmt = c(:i-1) // Trim(adjustL(cn)) // c(j+1:)
      End Function N_Fmt
```

```shell
sed -i 's/\/=\.TRUE\./\.neqv\.\.TRUE\./g' mod_scal.F
sed -i 's/==\.TRUE/\.eqv\.\.TRUE/g' internal_step.F
sed -i 's/==\.FALSE\./\.eqv\.\.FALSE\./g' adv_t.F
sed -i 's/==\.FALSE\./\.eqv\.\.FALSE\./g' adv_s.F
```

9. Install fvcom

```shell
make -j $(nproc)
```

8. Check whether the installation is successful:

If the fvcom file is generated, the installation is successful.


