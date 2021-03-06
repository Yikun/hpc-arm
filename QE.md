# QUANTUM ESPRESSO 6.7.0 Porting Guide(CentOS 8.0)
This is the distribution of the Quantum ESPRESSO suite of codes (ESPRESSO: opEn-Source Package for Research in Electronic Structure, Simulation, and Optimization)
Open-source protocol: public domain

## Environment Requirements
### Software Requirements
| Item  | Version  |  Download Address |
| ------------ | ----------- | ------------ |
|  OpenMPI          | 4.0.3  | https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz |
|  OpenBLAS         | 0.3.14 | https://github.com/xianyi/OpenBLAS/archive/refs/tags/v0.3.14.tar.gz |
|  Scalapack        | 2.1.0  | http://www.netlib.org/scalapack/scalapack-2.1.0.tgz |
|  Quantum ESPRESSO | 6.7.0  |  https://github.com/QEF/q-e/archive/refs/tags/qe-6.7.0.tar.gz |

### OS Requirements
| Item  | Version  | How to Obtain  |
| ------------ | ------------ | ------------ |
|  CentOS | 8.0  |  https://www.centos.org/download/ |
| Kernel  | 4.18.0-80  |  Included in the OS image. |

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

```shell
yum install -y centos-release-scl
yum install -y devtoolset-9-gcc
yum install -y devtoolset-9-gcc-c++
yum install -y devtoolset-9-gcc-gfortran
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
### Installing openblas
1. Run the following commands to install OpenBLAS:

```shell
wget -O OpenBLAS-0.3.14.tar.gz https://github.com/xianyi/OpenBLAS/archive/refs/tags/v0.3.14.tar.gz 
tar -zxvf OpenBLAS-0.3.14.tar.gz
cd OpenBLAS-0.3.14
export CC=`which gcc`
export CXX=`which g++`
export FC=`which gfortran`
make -j $(nproc)
make PREFIX=/path/to/OPENBLAS install
```

2. Configure environment variables:

```shell
export LD_LIBRARY_PATH=/path/to/OPENBLAS/lib:$LD_LIBRARY_PATH
```

### Installing Scalapack
1. Run the following commands to install Scalapack:

```shell
wget http://www.netlib.org/scalapack/scalapack-2.1.0.tgz
tar -xvf scalapack-2.1.0.tgz
cd scalapack-2.1.0
cp SLmake.inc.example SLmake.inc
vi SLmake.inc , and set:
BLASLIB       = -L/path/to/OPENBLAS/lib -lopenblas
LAPACKLIB     = -L/path/to/OPENBLAS/lib -lopenblas
make
mkdir -p /path/to/SCALAPACK
cp libscalapack.a /path/to/SCALAPACK
```

2. Configure environment variables:

```shell
export LD_LIBRARY_PATH=/path/to/SCALAPACK:$LD_LIBRARY_PATH
```

### Installing Quantum ESPRESSO
1. Run the following commands to install QE:

```shell
wget https://github.com/QEF/q-e/archive/refs/tags/qe-6.7.0.tar.gz
tar -zxvf qe-6.7.0.tar.gz
cd q-e-qe-6.7.0
export BLAS_LIBS="-L/path/to/OPENBLAS/lib -lopenblas"
export LAPACK_LIBS="-L/path/to/OPENBLAS/lib -lopenblas"
export SCALAPACK_LIBS="-L/path/to/SCALAPACK -lscalapack"
./configure F90=gfortran F77=gfortran MPIF90=mpifort MPIF77=mpifort CC=mpicc \
FCFLAGS="-O3" CFLAGS="-O3" \
--with-scalapack=yes \
--prefix=/path/to/QE
make -j $(nproc) all
make install
```

2. Configure environment variables:

```shell
export PATH=/path/to/QE/bin:$PATH
```

