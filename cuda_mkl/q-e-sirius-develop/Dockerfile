FROM nvidia/cuda:11.0-devel-ubuntu20.04

ENV DEBIAN_FRONTEND noninteractive

ENV FORCE_UNSAFE_CONFIGURE 1

RUN apt-get update && \
    apt-get install -y apt-utils gcc g++ gfortran git make cmake unzip \
    vim wget pkg-config python3-pip curl environment-modules tcl m4 \
    apt-transport-https ca-certificates gnupg software-properties-common

ARG MPICH_VERSION=3.1.4
ENV MPICH_VERSION ${MPICH_VERSION}

# get and build mpich
RUN wget https://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz && \
    tar -xzvf mpich-${MPICH_VERSION}.tar.gz && \
    cd mpich-${MPICH_VERSION} && \
    ./configure && \
    make install -j6 && \
    cd .. && \
    rm mpich-${MPICH_VERSION}.tar.gz && \
    rm -rf mpich-${MPICH_VERSION} && \
    ldconfig

ENV SPEC "sirius@develop target=haswell build_type=Release +fortran +scalapack +magma +cuda cuda_arch=60 ^mpich@$MPICH_VERSION ^libxc ^spla ^spfft+single_precision ^intel-mkl@2017: threads=openmp ^magma+cuda cuda_arch=60 ^cuda"

RUN git clone https://github.com/spack/spack.git && \
    /spack/bin/spack compiler find && \
    /spack/bin/spack external find --not-buildable cuda && \
    /spack/bin/spack external find --not-buildable mpich && \
    /spack/bin/spack install ${SPEC}

RUN cd && \
    git clone --recursive -b ristretto https://github.com/electronic-structure/q-e-sirius.git && \
    cd q-e-sirius && \
    mkdir build && \
    cd build && \
    export sirius_DIR=$(/spack/bin/spack location -i sirius) && \
    /spack/bin/spack build-env $SPEC -- cmake .. -DQE_ENABLE_OPENMP=1 -DQE_ENABLE_SIRIUS=1 -DQE_ENABLE_SCALAPACK=1 -DQE_ENABLE_LIBXC=1 && \
    /spack/bin/spack build-env $SPEC -- make pw -j && \
    cp /root/q-e-sirius/build/bin/pw.x /usr/bin


#ENTRYPOINT ["bash", "-l", "-c"]
