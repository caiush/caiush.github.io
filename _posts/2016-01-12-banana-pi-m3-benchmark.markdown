

We benchmarked the BPI M3 against the current 500 gorilla, the [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/). 

First up the weigh in. The raw stats for the two boards below. Note, just to get a sense of absolute scale I also include the specs of a Google Compute Engine [n1-standard-1](https://cloud.google.com/compute/docs/machine-types).

| Metric | Raspberry Pi 2 Model B | BPI M3 | n1-standard-1 |
|--------|-----------------------|--------|
| CPU Model  |  ARM Cortex-A7 | ARM A83T |  Haswell | 
| CPU Speed  |  900MHz | 2GHz | 2.30GHz |
| Core Count |  4  | 8 | 1 |
| RAM        |  1GB | 2GB | 3.75GB |
| Ethernet   | 100Mb/s | 1000Mb/s |  2000Mb/s |
| Cost  | [$37.61](http://amzn.to/1ROanST) | [$78](http://amzn.to/1ROaCNA) | [$0.035/hr](https://cloud.google.com/compute/pricing) | 
| Max Power Draw / W | 10 | 10 |  N/A | 

The current availability of the BPI M3 in the US is keeping the prices a bit high. Cheaper deals can be found on Chinese vendor sites, [for example](http://www.aliexpress.com/store/product/2GB-of-RAM-Octa-Core-BPI-M3-Banana-Pi-M3-Single-board-computer-development-board-with/302756_32532101730.html). 

## The Set Up

For the SD card I used [Samsung's 64GB EVO](http://amzn.to/1n1R38S) SD card in both the RPi2 and the M3. I've noticed, but havnt yet benched, a significant difference in performance based on the SD card I used. The EVO used to be the wirecutter recommended SD card, but they now favor the [EVO+](http://thewirecutter.com/reviews/best-microsd-card/). 

For the OS I went with the Raspian Jessie Lite Nove 2015 release [download](https://www.raspberrypi.org/downloads/raspbian/) for the RPi2 and Raspbian Jessie Lite Beta 1.0 release for the M3, [download](http://www.bananapi.com/index.php/download?layout=edit&id=90). Burning these images is [pretty easy](https://www.raspberrypi.org/documentation/installation/installing-images/).

One the boards are booted, time to install the bench tools. The first thing I discovered is there really isnt that many great benching tools out there. I used the following tests:
     1) NBench: a goldern oldy. Dead easy to compile and run, but only tests single treaded performance. 
     2) sysbench: I didnt bother compiling in the mysql bench marks.
     3) openssl tests. 

## NBench and single thread perfomance 

As the OS is the same on both boards the install of NBench is exactly the same:

{% highlight bash %}
sudo apt-get install build-essential wget
wget http://www.tux.org/~mayer/linux/nbench-byte-2.2.3.tar.gz
tar zxf nbench-byte-2.2.3.tar.gz
cd nbench-byte-2.2.3
time make
{% endhighlight %}


   