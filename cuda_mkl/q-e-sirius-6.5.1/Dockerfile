FROM nvidia/cuda:10.1-devel-ubuntu18.04

ENV DEBIAN_FRONTEND noninteractive

ENV FORCE_UNSAFE_CONFIGURE 1


ARG SPFFT_VERSION=0.9.10
ARG SIRIUS_VERSION=6.5.2
ARG QE_VERSION=6.5-rc4-sirius
ARG MPICH_VERSION=3.1.4
ENV MPICH_VERSION ${MPICH_VERSION}

RUN apt-get update

RUN apt-get install -y apt-utils

# install basic tools
RUN apt-get install -y gcc g++ gfortran git make unzip \
  vim wget pkg-config python3-pip curl environment-modules tcl \
  apt-transport-https ca-certificates gnupg software-properties-common \
  libhdf5-dev libgsl-dev libxc-dev

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
  -DSPFFT_MPI=ON -DSPFFT_OMP=ON -DCMAKE_INSTALL_PREFIX=/usr/local

RUN cd SpFFT-$SPFFT_VERSION/build && make -j12 install

# install SIRIUS
RUN wget https://github.com/electronic-structure/SIRIUS/archive/v$SIRIUS_VERSION.tar.gz && tar zxvf v$SIRIUS_VERSION.tar.gz

RUN cd SIRIUS-$SIRIUS_VERSION && CC=mpicc CXX=mpicxx FC=mpif90 FCCPP=cpp python3 prerequisite.py /usr/local spg

RUN mkdir SIRIUS-$SIRIUS_VERSION/build && cd SIRIUS-$SIRIUS_VERSION/build && LIBSPGROOT=/usr/local \
    cmake .. -DSpFFT_DIR=/usr/local/lib/cmake/SpFFT -DUSE_SCALAPACK=1 -DUSE_MKL=1 -DBUILD_TESTS=1 \
    -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DUSE_CUDA=On -DGPU_MODEL='P100'

RUN cd SIRIUS-$SIRIUS_VERSION/build && make -j12 install

ENV SIRIUS_BINARIES /usr/local/bin/

RUN wget https://github.com/electronic-structure/q-e-sirius/archive/v$QE_VERSION.tar.gz && tar zxvf v$QE_VERSION.tar.gz
#RUN git clone https://gitlab.com/QEF/q-e.git q-e-sirius-6.5-rc3-sirius


RUN cd q-e-sirius-6.5-rc4-sirius && ./configure --with-scalapack --enable-parallel

RUN cd q-e-sirius-6.5-rc4-sirius && \
    sed -i -e "s/^BLAS_LIBS\ *=\ *.*/BLAS_LIBS =/" make.inc && \
    sed -i -e "s/^LAPACK_LIBS\ *=\ *.*/LAPACK_LIBS =/" make.inc && \
    sed -i -e "s/LAPACK_LIBS_SWITCH = internal/LAPACK_LIBS_SWITCH = external/" make.inc && \
    sed -i -e "s/BLAS_LIBS_SWITCH = internal/BLAS_LIBS_SWITCH = external/" make.inc && \
    sed -i -e "s/^DFLAGS\ *=\ *.*/DFLAGS = -D__MPI -D__SCALAPACK -D__DFTI -I\/usr\/local\/include\/sirius /" make.inc && \
    sed -i -e "s/^LD_LIBS\ *=\ *.*/LD_LIBS = -L\/usr\/local\/lib -lsirius -Wl,-rpath,\/usr\/local\/lib -L\$(MKLROOT)\/lib\/intel64 -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_gnu_thread -lmkl_core -lmkl_blacs_intelmpi_lp64 -fopenmp/" make.inc && \
    sed -i -e "s/^FFLAGS\ *=\ *.*/FFLAGS = -march=core-avx2 -O3 -march=haswell -fopenmp -ftree-vectorize -fopt-info -fopt-info-missed -fopt-info-vec -fopt-info-loop /" make.inc && \
    make -j 12 pw

