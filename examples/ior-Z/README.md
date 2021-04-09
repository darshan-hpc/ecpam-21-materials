# `ior -Z` Example

Synopsis: investigate why a particular I/O benchmark configuration reports
different amounts of data written and read from the file.

* Date collected: 2021-04
* Executed on Theta system at the ALCF
* Application: ior 3.3
  * see ior-Z.qsub for command line: `ior -w -r -o $DATAFILE -c -Z -b 1000000 -t 1000000 -s 1 -a MPIIO`

## Analysis

There are a few ways to look at this and spot the oddity in this run.  If
you are using darshan-job-summary, compare the mpi-io and posix access size
histograms.  In the former, the r/w perfectly matches.  In the latter, they
do not.  Alternatively you can look at the `POSIX_BYTE_{READ|WRITTEN}` or
`MPIIO_BYTES_{READ|WRITTEN}` to see a different take on the same issue, or
use one to validate the other.

The log in this example has DXT enabled (anticipating that you need that to
really dig into the problem).  You can check a few different things, but the
smoking gun is in the offsets written and read at the MPI-IO level.  The
offsets read include some duplicates:

darshan-dxt-parser carns_ior_id510260_4-6-49778-9578554294911421599_1617716983.darshan |grep MPIIO |grep read | awk -e '{print $5}' |sort -n |uniq -c |head
      1 0
      2 1000000
      3 6000000
      1 7000000
      1 8000000
      1 10000000
      4 11000000
      1 12000000
      2 14000000
      3 15000000

A python script or jupyter notebook that plotted the offsets over time from
dxt would probably illustrate this.  You can't see it in the dxt_analyzer.py
output, though.

## Conclusion

ior is independently randomizing what portion of the file to read at each
rank.  Inevitably some ranks read the same portion of the file as other
ranks.

These duplicate reads can be serviced from the same collective buffer in
ROMIO, meaning that ROMIO (rightly) never bothers to read some portions of
the file to service this workload.  Thus the bytes transferred imbalance.

The meta finding is that ior -Z probably isn't what you want if your goal is
to put more stress on a storage system or defeat caching.  The following is
the documentation of this option in the ior manual, but it doesn't indicate
if the randomization is independent or shuffled across all ranks.

"-Z reorderTasksRandom -- changes task ordering to random ordering for
readback".
