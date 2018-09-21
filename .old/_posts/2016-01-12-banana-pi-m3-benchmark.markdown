---
layout: post
title:  "Benchmarking the Banana Pi M3"
date:   2016-01-16 16:27:43
categories: raspberrypi
---

We benchmarked the [Banana Pi M3](http://www.banana-pi.org/m3.html) against a stock [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/). 

The raw stats for the two boards below. Note, just to get a sense of absolute scale I also include the specs of a Google Compute Engine [n1-standard-1](https://cloud.google.com/compute/docs/machine-types).

| Metric | Raspberry Pi 2 Model B | BPI M3 | n1-standard-1 |
|--------|-----------------------|--------|
| CPU Model  |  ARM Cortex-A7 | ARM A83T |  Haswell | 
| CPU Speed  |  900MHz | 2GHz | 2.30GHz |
| Core Count |  4  | 8 | 1 |
| RAM        |  1GB | 2GB | 3.75GB |
| Ethernet   | 100Mb/s | 1000Mb/s |  2000Mb/s |
| Cost  | [$37.61](http://amzn.to/1ROanST) | [$78](http://amzn.to/1ROaCNA) | [$0.035/hr](https://cloud.google.com/compute/pricing) | 
| Max Power Draw / W | 10 | 10 |  N/A | 


The current availability of the BPI M3 in the US is keeping the prices a bit high. Cheaper deals can be found on Chinese vendor sites, for example [AliExpress](http://www.aliexpress.com/store/product/2GB-of-RAM-Octa-Core-BPI-M3-Banana-Pi-M3-Single-board-computer-development-board-with/302756_32532101730.html). 

## The Set Up

For the SD card I used [Samsung's 64GB EVO](http://amzn.to/1n1R38S) SD card in both the RPi2 and the M3. I've noticed, but haven't yet benched, a significant difference in performance based on the SD card I used. The EVO used to be the wirecutter recommended SD card, but they now favor the [EVO+](http://thewirecutter.com/reviews/best-microsd-card/). 

For the OS I went with the Raspian Jessie Lite Nove 2015 release [download](https://www.raspberrypi.org/downloads/raspbian/) for the RPi2 and Raspbian Jessie Lite Beta 1.0 release for the M3, [download](http://www.bananapi.com/index.php/download?layout=edit&id=90). Burning these images is [pretty easy](https://www.raspberrypi.org/documentation/installation/installing-images/).

Once the boards are booted time to install some benching tools. The first thing I discovered is there really isnt that many great benching tools out there for arm. I used the following tests:
     1) NBench: a goldern oldy. Dead easy to compile and run, but only tests single treaded performance. 
     2) sysbench: I didnt bother compiling in the mysql bench marks.

## NBench and Single Thread Performance

As the OS is the same on both boards the install of NBench is exactly the same:

{% highlight bash %}
sudo apt-get install build-essential wget
wget http://www.tux.org/~mayer/linux/nbench-byte-2.2.3.tar.gz
tar zxf nbench-byte-2.2.3.tar.gz
cd nbench-byte-2.2.3
time make
./nbench
{% endhighlight %}

I did the same thing on a Haswell GCE instance, just to get an idea of what "fast" is. As stated above, NBench only really measures single core performance, but its still interesting. Ndench reports its results in terms of number of iterations per second, so *larger numbers are better!*

| Metric | RBPi-M3  | RPi2 | GCE | RPi2/BPi-M3 | CGE/BPi-M3 | 
| -------| ----- | -------| -----|
| NUMERIC SORT | 792	 | 404	| 1701	| 0.5 |	2.1
| STRING SORT |	73	| 47| 	1175| 	0.7	| 16.1 |
| BITFIELD	| 214 M	| 118 M	| 630M	| 0.5	| 2.9 |
| FP EMULATION|	159	| 58	| 701| 0.4	| 4.4 |
| FOURIER |	7652	| 3980	| 50111	| 0.5	| 6.5 | 
| ASSIGNMENT	| 13.4	| 7.6	| 68.3	| 0.6	| 5.2 |
| IDEA	| 2472	| 1570	| 12483	|0.6	|5.0 |
| HUFFMAN	| 1294	| 690	| 5903	| 0.5	| 4.6 |
| NEURAL NET |11.0 | 5.78 | 123	| 0.5	| 11.2 |
| LU DECOMPOSITION	| 423 |	236	| 3370 | 0.6 | 8.0 |


There a few things worth noting here. Firstly, as expected the BPi-M3 is about twice as fast as the stock RPi2. The chips are similar, the M3 is just running twice as fast. However, when we compare against the virtualized Xeon E5's in the GCE instance we see, as expected, considerably more variation in performance depending on the type of calculation. 

NBench only tests a single core. As the BPiM3 has twice the core count and each core is about twice as fast, we should be better off by a factor of 4 over the RPi2. So lets test multicore performance with sysbench.

# Sysbench and Multicore Performance 

Sysbench can run N treads computing prime numbers and times 10K primes. Compiling sysbench:
{% highlight bash %}
sudo apt-get install libtool automake
# single threaded
./sysbench/sysbench  --test=cpu --num-threads=1 run
# 4 threaded
./sysbench/sysbench  --test=cpu --num-threads=4 run
{% endhighlight %}

We compute the number of primes per second in both single threaded and all threads: 4, 8 and 1 for the RPi2, BPiM3 and GCE respectively.

| threading | Bpi M3 | RPi2 | GCE | 
|-----------|---------|------|-----|
| Single (primes/sec) |68.5	 | 33.7	| 943.4 | 
| Multi (primes/sec) | 403.2 | 133.3 | 943.4 | 

As expected the the Bpi-M3 is about 4 times as fast as the RPi2 and (for this computation) about half the speed of the GCE instance, which when you consider the cost of the BPiM3, is really quite a great deal. 

# The Cost of Computing Primes

Just for fun, I compute the cost of calculating 1e9 prime numbers. I assume that the you are running the computations for 3 years and the cloud instance would also be up the entire 24x7. I assume that the running costs of a single board is twice that we pay for energy.

| vendor | Bpi M3 | RPi2 | GCE | 
|-----------|---------|------|-----| 
| Cost $ / 1G primes | 3.12	 | 4.40	| 7.5  |

Thats interesting...

