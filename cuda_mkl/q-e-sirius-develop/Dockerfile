FROM nvidia/cuda:11.0-devel-ubuntu20.04

ENV DEBIAN_FRONTEND noninteractive

ENV FORCE_UNSAFE_CONFIGURE 1

RUN apt-get update

RUN apt-get install -y apt-utils

# install basic tools
RUN apt-get install -y gcc g++ gfortran git make cmake unzip \
  vim wget pkg-config python3-pip curl environment-modules tcl mpich \
  apt-transport-https ca-certificates gnupg software-properties-common

# get latest version of spack
RUN cd && git clone https://github.com/spack/spack.git

ENV SPACK_ROOT /root/spack

ENV SPEC "sirius@develop target=haswell build_type=Release +fortran +scalapack +magma +cuda cuda_arch=60 ^mpich@3.3.2 ^libxc ^spla ^spfft+single_precision ^intel-mkl@2017: threads=openmp ^magma+cuda cuda_arch=60 ^cuda"

# add environment variables
RUN echo "source /root/spack/share/spack/setup-env.sh" >> /etc/profile.d/spack.sh

# update list of spack compilers
RUN /bin/bash -l -c "spack compiler find"
# add CUDA
RUN /bin/bash -l -c "spack external find --not-buildable cuda"

RUN /bin/bash -l -c "spack install --only=dependencies $SPEC"

RUN /bin/bash -l -c "spack install $SPEC"

RUN cd && git clone --recursive -b ristretto https://github.com/electronic-structure/q-e-sirius.git && \
    cd q-e-sirius && mkdir build

RUN /bin/bash -l -c "cd /root/q-e-sirius/build && spack load m4 && spack load sirius && spack build-env $SPEC -- cmake .. -DQE_ENABLE_OPENMP=1 -DQE_ENABLE_SIRIUS=1 -DQE_ENABLE_SCALAPACK=1 -DQE_ENABLE_LIBXC=1"

RUN /bin/bash -l -c "cd /root/q-e-sirius/build && spack build-env $SPEC -- make pw -j"

RUN cp /root/q-e-sirius/build/bin/pw.x /usr/bin

#ENTRYPOINT ["bash", "-l"]
