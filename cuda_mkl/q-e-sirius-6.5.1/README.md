To build a container:
``` bash
sudo docker build -t cuda_mkl/q-e-sirius-6.5.1 .
```

To save into a file:
``` bash
sudo docker save --output qe_sirius.tar cuda_mkl/q-e-sirius-6.5.1
```

To load image to Sarus:
``` bash
sarus load qe_sirius.tar qe_sirius
```

To run the container:
``` bash
OMP_NUM_THREADS=12 MKL_NUM_THREADS=12 srun -C gpu --partition=debug -N2 -n2 -c12 --hint=nomultithread sarus run --mount=type=bind,source=$SCRATCH,destination=$SCRATCH --mpi load/library/qe_sirius:latest bash -c 'cd $SCRATCH/Si63Ge-scf+forces && /root/q-e-sirius-6.5-rc3-sirius/PW/src/pw.x -i pw.in -ndiag 1 -npool 2 -sirius'
```
