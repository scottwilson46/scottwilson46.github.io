---
layout: post
title:  "Mercenne Twister on an FPGA"
date:   2014-09-06 21:11:52
categories: FPGA random PRBS Mercenne Twister
---

So now we have established a way of generating random numbers on an FPGA and testing their uniformity, lets look at a better uniform random number generation algorithm, Mercenne Twister.  
There is C-code provided for this here:

<http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/emt.html>

I did a fairly naive implementation on this for targetting an FPGA and after the initialization phase (which only happens once after reset), 
it generates a random number every 3 cycles.

The way the algorithm works on FPGA is that we have a RAM of 624 elements and we iterate over this RAM and read-modify-writing the values with
new calculated values usually based on the ith and i+1th element.

The verilog code can be found here:

<http://github.com/scottwilson46/FPGARandom>

### Simulation

The testbench used in the previous post:

<http://scottwilson46.github.io/fpga/random/prbs/2014/07/06/fpga-random-prbs/>

was modified to use the mercenne twister code rather than the PRBS.

To run the simulation yourself, you will need to install both icarus verilog and gtkwave:

<http://iverilog.icarus.com/home>

and

<http://gtkwave.sourceforge.net/>

You can then use the run script in the verilog directory to set off the simulation (source run_mercenne_sim).

When we run the simulation, we can see that the calculated value for pi is a lot closer to what we expect it to be than using the PRBS version which would
suggest that the mercenne twister random number generator is a lot more uniform.

### Timing

So, again lets compare the performance between the naive FPGA implementation and the CPU version.  I modified the c-code in:

<http://github.com/scottwilson46/FPGARandom>

and instead of using the PRBS random number generator, pulled in the mercenne twister source code and used the function:

{% highlight c %}
genrand_int31() 
{% endhighlight %}

instead.  To compile it yourself, download the mercenne twister source code from here:

<http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/MT2002/emt19937ar.html>

and compile using:

gcc mt19937ac.c calc_pi_mercenne.c

For 1,000,000 iterations of the monte-carlo simulation on a CPU (using the 1.7GHz Intel Core i7 on my Macbook Air) the time comes out as around 40ms.

For the FPGA version, synthesizing the code in Quartus gives us a maximum frequency of 90MHz (again the multipliers are the slowest paths).

THis would give us (assuming 1 iteration every 3 cycles):

(1/30e6) * 1e6 * 1e3 = 33ms

So the FPGA implementation is still faster, however it could be made faster still by pipelining the initialization part of the algorithm because
thats the only bit that uses multipliers.  

Added bonus:

Another pretty good uniform random number generator is the Tausworth Generator.  I created some verilog code for this too (in the same repo).
The advantage of this method is it is a lot less comutationally complex than using Mercenne Twister so the FPGA can again run at increased 
frequency and it generates a new random number every cycle so the speed for 1,000,000 iterations would be:

1/150e6 * 1e6 * 1e3 = 6.7ms

I also did a software version of this too (in the c directory which took 28ms to run so again the FPGA version is over 4 times faster.







