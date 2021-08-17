# GROMACS 2021.2 Porting Guide(CentOS 7.6)

## Introduction

## Environment Requirements

### Software Requirements

| Item     | Version | Download Address                                             |
| -------- | ------- | ------------------------------------------------------------ |
| OpenMPI  | 4.0.3   | https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz |
| FFTW3    | 3.3.9   | https://www.fftw.org/fftw-3.3.9.tar.gz                       |
| OpenBLAS | 0.3.15  | https://github.com/xianyi/OpenBLAS/releases/download/v0.3.15/OpenBLAS-0.3.15.tar.gz |
| CMake    | 3.21.1  | https://github.com/Kitware/CMake/releases/download/v3.21.1/cmake-3.21.1.tar.gz |
| GROMACS  | 2021.2  | https://ftp.gromacs.org/gromacs/gromacs-2021.2.tar.gz        |

### OS Requirements

| Item    | Version    | How to Obtain                    |
| ------- | ---------- | -------------------------------- |
| Cent OS | 7.6        | https://www.centos.org/download/ |
| Kernel  | 4.14.0-115 | Included in the OS image         |

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

1. Install the system dependency package:


```shell
yum install libxml2* systemd-devel.aarch64 numa* -y
```

2. Install Open MPI:


```shell
wget https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz
tar -zxvf openmpi-4.0.3.tar.gz
cd openmpi-4.0.3
./configure --prefix=/path/to/OPENMPI --enable-pretty-print-stacktrace --enable-orterun-prefix-by-default  --with-cma --enable-mpi1-compatibility
make -j $(nproc) all
make install
```

3. Configure environment variables:


```shell
export PATH=/path/to/OPENMPI/bin:$PATH
export LD_LIBRARY_PATH=/path/to/OPENMPI/lib:$LD_LIBRARY_PATH
```

### Installing FFTW3

1. Install FFTW3:

```shell
wget https://www.fftw.org/fftw-3.3.9.tar.gz
tar -zxvf fftw-3.3.9.tar.gz
cd fftw-3.3.9
make clean
./configure --enable-neon --enable-shared --enable-float --prefix=/path/to/FFTW
make -j 16
make install
```

2. Configure environment variables

```shell
export LD_LIBRARY_PATH=/path/to/FFTW/lib:$LD_LIBRARY_PATH
```

### Installing OpenBLAS

1. Install OpenBLAS

```shell
wget https://github.com/xianyi/OpenBLAS/releases/download/v0.3.15/OpenBLAS-0.3.15.tar.gz
tar -zxvf OpenBLAS-0.3.15.tar.gz
cd OpenBLAS-0.3.15
make
make PREFIX=/path/to/OPENBLAS install
```

2. Configure environment variables

```shell
export LD_LIBRARY_PATH=/path/to/OPENBLAS/lib:$LD_LIBRARY_PATH
```

### Installing CMake

1. Install CMake

```shell
wget https://github.com/Kitware/CMake/releases/download/v3.21.1/cmake-3.21.1.tar.gz
tar -zxvf cmake-3.21.1.tar.gz
cd cmake-3.21.1
./configure --prefix=/path/to/CMAKE
make
make install
```

2. Configure environment variables

```shell
export PATH=/path/to/CMAKE/bin:$PATH
```

## Compiling and installing GROMACS

Get the source code

```shell
wget https://ftp.gromacs.org/gromacs/gromacs-2021.2.tar.gz
tar -zxvf gromacs-2021.2.tar.gz
cd gromacs-2021.2
```

Create and change to the build directory

```shell
mkdir build
cd build
```

Configure the cmake options and install

```shell
FLAGS="-mcpu=tsv110"; CFLAGS=$FLAGS CXXFLAGS=$FLAGS LDFLAGS="-lgfortran" CC=mpicc CXX=mpicxx \
cmake \
-DCMAKE_INSTALL_PREFIX=/path/to/INSTALL \
-DBUILD_SHARED_LIBS=on \
-DBUILD_TESTING=on \
-DREGRESSIONTEST_DOWNLOAD=on \
-DGMX_BUILD_OWN_FFTW=off \
-DGMX_SIMD=ARM_NEON_ASIMD \
-DGMX_DOUBLE=off \
-DGMX_EXTERNAL_BLAS=on \
-DGMX_EXTERNAL_LAPACK=on \
-DGMX_FFT_LIBRARY=fftw3 \
-DGMX_BLAS_USER= /path/to/OPENBLAS/lib/libopenblas.a \
-DGMX_LAPACK_USER=/path/to/OPENBLAS/lib/libopenblas.a \
-DFFTWF_LIBRARY=/path/to/FFTW/lib/libfftw3f.so \
-DFFTWF_INCLUDE_DIR=/path/to/FFTW/include/ \
-DGMX_GPU=off \
-DGMX_MPI=on \
-DGMX_OPENMP=off \
-DGMX_X11=off \
-DREGRESSIONTEST_DOWNLOAD=OFF \
../
```

Run the environment script

```shell
source /path/to/INSTALL/bin/GMXRC
```

Now your program is ready to run.
