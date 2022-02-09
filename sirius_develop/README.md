Build the container:
```bash
sudo docker build -t sirius_develop .
```

Run the container:
```bash
sudo docker run -i -t sirius_develop
```

Inside container run to build a simple test:
```bash
spack -e ci build-env $SIRIUS_SPEC -- mpif90 test.f90 -I$SIRIUS_DIR/include/sirius -L$SIRIUS_DIR/lib -lsirius -Wl,-rpath,$SIRIUS_DIR/lib
./a.out
 ```
Expectd output:
```bash
SIRIUS 7.3.1, git hash: 02294f8070497a10564e50304b042f56beaf0dcc
Warning! Compiled in 'debug' mode with assert statements enabled!
```

To build your code, prepend the `make` command with `spack -e ci build-env $SIRIUS_SPEC -- `
