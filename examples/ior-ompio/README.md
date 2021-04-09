# `ior ompio` Example

Synopsis: investigate why a particular I/O benchmark configuration that it
opened way more files than requested.

* Date collected: 2021-04
* Executed on laptop
* Application: ior 3.3, with OpenMPI 4.0.5
  * `mpiexec -n 4 ior -w -r -a MPIIO -o foo.dat`

## Analysis

The first log is run with the above command line and no special options to
`mpiexec`.  The ior program is instructed to open a single file, `foo.dat`.
If you produce a Darshan job summary, however, it reports that 8 POSIX files
were opened.

Why?

There are a few ways to get the list of files out of the Darshan log.  One
simple way is with `darshan-parser --file-list <log>`.  An exerpt is shown
below:

```
2525422531822774723	/home/carns/foo.dat.locktest.0	1	0.000193	0.000193
10942178388105573093	/tmp/ompi.carns-x1-7g.1000/pid.1002285/1/foo.dat_cid-1-1002290.sm	1	0.000016	0.000016
2173836627065155674	/home/carns/foo.dat	4	0.002275	0.001888
10323258089345963197	/tmp/ompi.carns-x1-7g.1000/pid.1002285/1/foo.dat_cid-4-1002290.sm	4	0.000072	0.000030
7145126889316099076	/home/carns/foo.dat.locktest.1	1	0.000180	0.000180
10204686826717149174	/home/carns/foo.dat.locktest.2	1	0.000117	0.000117
17520034237240197387	/home/carns/foo.dat.locktest.3	1	0.000210	0.000210

...

15920181672442173319	<STDOUT>	1	0.000028	0.000028
```

The `<SDTDOUT>` entry is just for the terminal output from the benchmark.

The other unexpected files are `*.sm` and `*.locktest.*` files, which look
like they may have been implicitly accessed by the middleware.

We can test this by rerunning the benchmark, this time instructing Open MPI
to use the ROMIO backend (rather than it's default ompio MPI-IO
implementation).  This can be done by adding `--mca io romio321` to the
`mpiexec` command line.  The second log shows this results.

You can do the same `darshan-parser --file-list` command on this second log
and see that the unexpected files are not there.  This confirms that these
were not explicitly touched by the application binary itself.

## Conclusion

Unknown if there are any performance implications, but the takeaway here is
that auxiliary libraries might perform I/O that wasn't expected based on the
application code.
