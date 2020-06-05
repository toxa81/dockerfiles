FROM nvidia/cuda:10.1-devel-ubuntu18.04

ENV DEBIAN_FRONTEND noninteractive

ENV FORCE_UNSAFE_CONFIGURE 1


ARG SPFFT_VERSION=0.9.10
ARG SIRIUS_VERSION=6.5.3
ARG QE_VERSION=6.5-rc4-sirius
ARG MPICH_VERSION=3.1.4
ENV MPICH_VERSION ${MPICH_VERSION}

RUN apt-get update

RUN apt-get install -y apt-utils

# install basic tools
RUN apt-get install -y gcc g++ gfortran git make unzip \
  vim wget pkg-config python3-pip curl environment-modules tcl \
  apt-transport-https ca-certificates gnupg software-properties-common \
  libhdf5-dev libgsl-dev

## install GCC-8
#RUN apt-get install -y gcc-8 g++-8 gfortran-8
#RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 40
#RUN update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 40
#RUN update-alternatives --install /usr/bin/gfortran gfortran  /usr/bin/gfortran-8 40

# install latest CMake
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | apt-key add -
RUN apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main'
RUN apt-get update
RUN apt-get install -y cmake

# get and build mpich
RUN wget https://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz
RUN tar -xzvf mpich-${MPICH_VERSION}.tar.gz
RUN cd mpich-${MPICH_VERSION} && \
    ./configure && \
    make install -j6
RUN rm mpich-${MPICH_VERSION}.tar.gz
RUN rm -rf mpich-${MPICH_VERSION}

# install MKL
RUN wget -O - https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB 2>/dev/null | apt-key add -
RUN apt-add-repository 'deb https://apt.repos.intel.com/mkl all main'
#RUN apt-add-repository 'deb https://apt.repos.intel.com/mpi all main'

RUN apt-get install -y intel-mkl-2020.0-088
#RUN apt-get install -y intel-mpi-2019.6-088
#RUN apt-get install -y intel-mpi-2019.1-053

#ENV PATH="/opt/intel/compilers_and_libraries/linux/mpi/intel64/bin:${PATH}"
#ENV PATH="/opt/intel/impi/2019.1.144/intel64/bin:${PATH}"

ENV MKLROOT=/opt/intel/compilers_and_libraries/linux/mkl

#RUN echo "/opt/intel/lib/intel64 \n/opt/intel/compilers_and_libraries/linux/mkl/lib/intel64 \n/opt/intel/compilers_and_libraries/linux/mpi/intel64/lib \n/opt/intel/compilers_and_libraries/linux/mpi/intel64/lib/release \n/opt/intel/compilers_and_libraries/linux/mpi/intel64/libfabric/lib/ \n/opt/intel/compilers_and_libraries/linux/mpi/intel64/libfabric/lib/prov/" >> /etc/ld.so.conf.d/intel.conf
#RUN echo "/opt/intel/lib/intel64 \n/opt/intel/compilers_and_libraries/linux/mkl/lib/intel64 \n/opt/intel/impi/2019.1.144/intel64/lib/release \n/opt/intel/impi/2019.1.144/intel64/lib/ \n/opt/intel/impi/2019.1.144/intel64/libfabric/lib/prov/ \n/opt/intel/impi/2019.1.144/intel64/libfabric/lib/" >> /etc/ld.so.conf.d/intel.conf
RUN echo "/opt/intel/lib/intel64 \n/opt/intel/compilers_and_libraries/linux/mkl/lib/intel64 \n/" >> /etc/ld.so.conf.d/intel.conf
RUN ldconfig

WORKDIR /root

# install SpFFT
RUN wget https://github.com/eth-cscs/SpFFT/archive/v$SPFFT_VERSION.tar.gz && tar zxvf v$SPFFT_VERSION.tar.gz

RUN mkdir SpFFT-$SPFFT_VERSION/build && cd SpFFT-$SPFFT_VERSION/build && \
  cmake .. -DCMAKE_BUILD_TYPE=RELEASE -DSPFFT_GPU_BACKEND=CUDA -DSPFFT_SINGLE_PRECISION=ON \
  -DSPFFT_MPI=ON -DSPFFT_OMP=ON -DCMAKE_INSTALL_PREFIX=/usr

RUN cd SpFFT-$SPFFT_VERSION/build && make -j12 install

# install SIRIUS
RUN wget https://github.com/electronic-structure/SIRIUS/archive/v$SIRIUS_VERSION.tar.gz && tar zxvf v$SIRIUS_VERSION.tar.gz

RUN cd SIRIUS-$SIRIUS_VERSION && CC=mpicc CXX=mpicxx FC=mpif90 FCCPP=cpp python3 prerequisite.py /usr spg xc

RUN mkdir SIRIUS-$SIRIUS_VERSION/build && cd SIRIUS-$SIRIUS_VERSION/build && LIBSPGROOT=/usr \
    cmake .. -DSpFFT_DIR=/usr/lib/cmake/SpFFT -DUSE_SCALAPACK=1 -DUSE_MKL=1 -DBUILD_TESTS=1 \
    -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DUSE_CUDA=On -DGPU_MODEL='P100'

RUN cd SIRIUS-$SIRIUS_VERSION/build && make -j12 install

ARG GPU_VERSION="P100"

ARG LIBINT_LMAX="5"

# build CP2K dependencies using the toolchain script
RUN git clone --recursive --single-branch -b master https://github.com/cp2k/cp2k.git

RUN apt-get install -y libfftw3-dev libfftw3-mpi-dev
ENV CUDA_PATH "/usr/local/cuda"
ENV LD_LIBRARY_PATH "${CUDA_PATH}/lib64/stubs:${CUDA_PATH}/lib64:${LD_LIBRARY_PATH}"

RUN cd cp2k/tools/toolchain            \
# check for mpicxx provided by Intel MPI instead of mpic++ provided by MPICH
    && sed -i 's/mpic++/mpicxx/' scripts/install_mpich.sh \
    && ./install_cp2k_toolchain.sh     \
         --with-cmake=__SYSTEM__       \
         --mpi-mode=mpich              \
         --math-mode=mkl               \
         --with-cosma=no               \
         --with-elpa=no                \
         --with-fftw3=__SYSTEM__       \
         --with-gsl=__SYSTEM__         \
         --with-hdf5=__SYSTEM__        \
         --with-libxc=__SYSTEM__       \
         --with-reflapack=no           \
         --with-scalapack=no           \
         --with-sirius=__SYSTEM__      \
         --with-spfft=__SYSTEM__       \
         --libint-lmax="${LIBINT_LMAX}"\
         --gpu-ver="${GPU_VERSION}"    \
    && rm -rf ./build \
# add SIRIUS include path in "local_cuda.psmp"
    && sed -i -e "s#include/cuda#usr/include/sirius#" install/arch/local_cuda.psmp

# Fix for Python3
RUN ln -s $(which python3) /bin/python

# build CP2K
RUN cd cp2k \
    && cp tools/toolchain/install/arch/local_cuda.psmp arch \
    && /bin/bash -c "source tools/toolchain/install/setup" \
    && make -j 20 ARCH=local_cuda VERSION="psmp"
    #&& rm -rf lib obj

## add folder with CP2K executables to PATH
#ENV PATH /usr/local/src/cp2k/exe/local_cuda:$PATH

