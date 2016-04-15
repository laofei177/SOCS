# SOCS
Self-Organized Criticality Sonification

The main part of the `SOCS` system is a [Pure Data](http://puredata.info) patch. The first three
channels play audio files in a continuous loop. The loop playing functionality is taken 
from Andy Farnell's [Designing Sound](https://mitpress.mit.edu/books/designing-sound) 
code examples as is the synthesized rain sound used in channel 4 of the patch. 

The Pure Data patch connects to a network port to listen to data being transmitted
by a custom Python script, `trafficSender.py` which takes data from CSV file of traffic 
data. When working from a dump file it actually first creates a pickle file which is then 
used to calculate the log returns.

Usage as follows:

    optional arguments:
      -h, --help            show this help message and exit
      -t TIME, --time TIME  Time interval between messages in s. Default is 1.0 s.
      -w WALK, --walk WALK  Slow down to walking pace (1 s intervals) at record
                            number indicated by this argument. Default is
                            sys.maxsize which means never slow down.
      -f FIRST, --first FIRST
                        Ignore all records before 'first'. Default is 1.
      -l LAST, --last LAST  Ignore all recordas after 'last'. Default is
                            sys.maxsize.
      -a, --absolute        Use absolute values of log returns. Default is signed
                            log returns.
      -s, --square          Square the log return values. Useful when listening to
                            data at high speed to make spikes more salient. If the
                            log return is negative this argument will negate the
                            square. E.g., -2 becomes -4.
                            
The `first` and `last` arguments are useful for rendering only a section of a traffic
dump file. For example:

>`python trafficSender.py -t 0.02 -f 2700 -l 5000`

would send traffic records 2700 through 5000 to the PD patch. the `-t` argument sets
the playback rate to 20 ms. This is useful for spooling more quickly through a large data
set. However, when running at high speed it is recommended to use the `-a` argument
which gives the absolute value of all log returns. Although this means that both positive 
and negative changes both result in increases in amplitude and filter response, the reduction 
in amplitude that results from negative log returns is more easily missed when playing back
at high speed. If your traffic data were sampled at, say, 1 s intervals, then when playing
back at 1 s it is recommended not to use the `-a` argument. The 'gaps' in the sound field
caused by negative log returns becomes quite noticeable and allow the listener to 
differentiate between big positive and negative shifts in system state. The following
will run the system at 1 s intervals with signed log returns being sent to the PD patch:

>`python trafficSender.py -t 1 -f 2700 -l 5000`

Regardless of the `-a` argument, signed log returns are always sent for the combined
traffic variable as the lower plot of the PD patch will then show the positive and negative
shifts in network state.

>`python trafficSender.py -t 0.02 -s 2770`

This invokes the script with signed values at a playback interval of 20 ms, but tells
the script to slow down to an interval of 1 s when it reaches record 2770.

The sample pickle file, `logreturns.pickle` was generated from  the 
`bd-1015440483-eth1-interpolated-event.csv` dump file available from Imperial College's 
[Quaint project](http://www.doc.ic.ac.uk/~uh/QUAINT/data/).

The script calculates the log return for the number of bytes and packets sent and received
in the specified time interval and bundles them into a single message which is transmitted
via OSC to the network port.

Make sure the PD abstractions in the `abstractions` folder are either copied into your PD
system path or else the folder itself is added to your PD system path.