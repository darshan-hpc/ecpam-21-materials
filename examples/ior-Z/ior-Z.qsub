#!/bin/bash
#COBALT -n 8
#COBALT -t 45
#COBALT --mode script
#COBALT -A CSC250STDM12
#COBALT -q debug-cache-quad
#COBALT -M carns@mcs.anl.gov

DATAFILE=/projects/CSC250STDM12/carns/ior.dat

export DXT_ENABLE_IO_TRACE=1
aprun -n 512 -N 64 ./ior -w -r -o $DATAFILE -c -Z -b 1000000 -t 1000000 -s 1 -a MPIIO

