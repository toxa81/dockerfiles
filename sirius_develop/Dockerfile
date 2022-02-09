FROM stabbles/sirius-cpu

ENV SIRIUS_SPEC sirius@develop%gcc+apps+elpa+tests+vdwxc+fortran build_type=RelWithDebInfo ^mpich ^openblas%gcc

RUN spack -e ci install $SIRIUS_SPEC

RUN echo "export SIRIUS_DIR=$(spack location -i sirius@develop%gcc)" >> /etc/profile.d/sirius.sh

WORKDIR /root

COPY test.f90 /root

ENTRYPOINT ["bash", "-l"]
