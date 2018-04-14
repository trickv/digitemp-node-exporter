# digitemp-node-exporter
Scripts &amp; config to get [digitemp](https://www.digitemp.com/) (DS9097U reading from DS18B20 sensors) into prometheus's [node-exporter](https://github.com/prometheus/node_exporter).

# How it works

* This defines a systemd timer, which runs every 10 seconds, executing digitemp_DS9097U to collect temperature readings
* Output is placed into /var/lib/prometheus/node-exporter/digitemp_DS9097U.prom in the node-exporter's ```textfile``` format.
* The [node-exporter](https://github.com/prometheus/node_exporter) then picks up the output and adds it to the metrics output

The default configuration for ```prometheus-node-exporter``` on Debian 9 includes the textfile module automatically and requires no customization for this to work.

# Getting started

This has been tested only on Debian 9 (on a Raspberry Pi, any arch will work).  The paths may be specific to the Debian ```prometheus-node-exporter``` package.

Prerequisites:
* Make sure the ```node-exporter``` is working.  ```curl http://localhost:9100/metrics```
* Make sure you have digitemp_DS9097U working already.  You should get and output like:
```
trick@jlh:~/digitemp-node-exporter $ sudo digitemp_DS9097U -c /etc/digitemp.conf -a
DigiTemp v3.7.1 Copyright 1996-2015 by Brian C. Lane
GNU General Public License v2.0 - http://www.digitemp.com
Apr 14 11:27:56 Sensor 0 C: -15.19 F: 4.66
Apr 14 11:27:57 Sensor 1 C: -17.00 F: 1.40
```

## Installation

There isn't a real one yet.  I just symlink the files into ```/etc/systemd/system/```.  For example:
```
git clone https://github.com/trickv/digitemp-node-exporter
cd digitemp-node-exporter
find digitemp-node-exporter.* | xargs realpath | xargs -n 1 -I {} sudo ln -s /etc/systemd/system/ {}
sudo systemctl daemon-reload
sudo systemctl enable digitemp-node-exporter.timer
sudo systemctl start digitemp-node-exporter.timer
```

You can then inspect that the timer is running:
```
sudo systemctl list-timers digitemp-node-exporter.timer
NEXT                         LEFT    LAST                         PASSED UNIT                         ACTIVATES
Sat 2018-04-14 11:43:00 UTC  1s left Sat 2018-04-14 11:42:55 UTC  3s ago digitemp-node-exporter.timer digitemp-node-exporter.service
```

...and see the textfile format in place:
```
trick@jlh:~/digitemp-node-exporter $ sudo cat /var/lib/prometheus/node-exporter/digitemp_DS9097U.prom
digitemp_DS9097U_sensor{address="2854D1BB040000E4"} -14.687500
digitemp_DS9097U_sensor{address="2896ACBA0400001B"} -16.500000
```

# node-exporter output example

In the ```node-exporter``` output it should look like this:
```
trick@jlh:~/digitemp-node-exporter $ curl -s http://localhost:9100/metrics | grep digitemp
# HELP digitemp_DS9097U_sensor Metric read from /var/lib/prometheus/node-exporter/digitemp_DS9097U.prom
# TYPE digitemp_DS9097U_sensor untyped
digitemp_DS9097U_sensor{address="2854D1BB040000E4"} -15.25
digitemp_DS9097U_sensor{address="2896ACBA0400001B"} -17.0625
node_textfile_mtime{file="digitemp_DS9097U.prom"} 1.5237051823351555e+09
```

# To Do

* Don't run timer as root
* Make a Debian package
* Set type of output data
* Support other 1-wire sensors? I don't have any other than DS18B20's

Pull requests / forks / taking this work as your own is very welcome.  I spent more time writing this readme than I did writing the timer :)
