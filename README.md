# GoveeBTTempLogger
Govee H5074, H5075, and H5177 Bluetooth Low Energy Temperature and Humidity Logger, and Govee H5182 and H5183 Smart Meat Thermometers

Uses libbluetooth functionality from BlueZ on linux to open the default Bluetooth device and listen for low energy advertisments from Govee H5074, H5075, H5177, H5182, H5183 thermometers.

Each of these devices currently cost less than $15 on Amazon and use BLE for communication, so don't require setting up a manufacterer account to track the data.  

GoveeBTTempLogger was initially built using Microsoft Visual Studio 2017, targeting ARM processor running on Linux. I'm using a Raspberry Pi 4 as my linux host. I've verified the same code works on a Raspbery Pi ZeroW and a Raspberry Pi 3b.

GoveeBTTempLogger creates a log file for each of the devices it receives broadcasted data from using a simple tab-separated format that's compatible with loading in Microsoft Excel. Each line in the log file has Date, Temperature, relative humidity, and battery percent. The log file naming format includes the unique Govee device name, the current year, and month. A new log file is created monthly.

## Major update to version 2.
Added the SVG output function, directly creating SVG graphs from internal data in a specified directory. The causes the program to take longer to start up as it will attempt to read all of the old logged data into an internal memory structure as it starts. Once the program has entered the normal running state it writes four SVG files per device to the specified directory every five minutes.

Here is an example filename: gvh-E35ECC215C0F-day.svg

![Image](./gvh-E35ECC215C0F-day.svg)

The most recent temperature and humidity are displayed in the vertical scale on the left. The temperature scale is displayed on the left side of the graph, the humidity scale on the right. The most recent time data is displayed in the top right, with a title on the top left of the graph.

Minimum and maximum temperature and humidity data, at the granularity of the graph, may be displayed. This is most useful in yearly graphs, where the granularity is one day. Here is the corresponding yearly graph for the previous daily graph: gvh-E35ECC215C0F-year.svg

![Image](./gvh-E35ECC215C0F-year.svg)

Humidity, and the humidity scale on the right, are automatically omitted if the current data reports a humidity of zero. The meat thermometer reports its current temperature and its alarm set temperature but no humidity measurement.

A simple text file mapping Bluetooth addresses to titles will be read from the filename gvh-titlemap.txt in the svg output directory.  Each line in the file should consist of the bluetooth address (in hexadecimal format with (`:`) between octets), whitespace, and the title. See [gvh-titlemap.txt](./gvh-titlemap.txt) for an example. If no title mapping exists, the Bluetooth address is used for the graph title.

If the --svg option is not added to the command line, the program should continue to operate exactly the same as it did before.

## Verbosity has been significantly changed since the intial release.

 * -v 0 no output to stdout. Errors still sent to stderr.
 * -v 1 prints all advertisments that have been decoded from Govee H5075, H5074, and H5177 thermometers to stdout.
 * -v 2 prints all advertisments recieved and categorized
 * -v levels higher than 2 print way too much debugging information, but can be interesting to look at.

## Prerequisites

### Linux

 * Kernel version 3.6 or above
 * ```libbluetooth-dev```

#### Ubuntu/Debian/Raspbian

```sh
sudo apt-get install bluetooth bluez libbluetooth-dev
make deb
sudo make install-deb
```

This will install a systemd unit `goveebttemplogger.service` which will automatically start GoveeBTTempLogger. The service can be configured using environment variables via
the `systemctl edit goveebttemplogger.service` command. By default, it writes logs to `/var/log/goveebttemplogger` and writes SVG files to `/var/www/html/goveebttemplogger`.

The following environment variables control the service:

* `VERBOSITY` controlls the verbosity level; default: `0`
* `LOGDIR` directory the TSV files are written to; default: `/var/log/goveebttemplogger`
* `TIME` Sets the frequency data is written to the logs; default: `60`
* `SVGARGS` controlls options for writing SVG files; default: `--svg /var/www/html/goveebttemplogger/ --battery 8 --minmax 8`
* `EXTRAARGS` can be used to pass extra arguments; default is unset (empty)

As an example, to disable SVG files, increase verbosity, and change the directory the TSV files are written to, use 
`sudo systemctl edit goveebttemplogger.service` and enter the following file in the editor:

```
[Service]
Environment="VERBOSITY=1"
Environment="LOGDIR=/opt/govee/data"
Environment="SVGARGS="
```

Then use `sudo systemctl restart goveebttemplogger` to restart GoveeBTTempLogger.

#### Windows Subsystem for Linux (WSL) Cross Compile requirements (Debian)

The first two commands below set up the required environment for Visual Studio 2022 to build the project. The third command added the required libraries to build bluetooth projects.

```sh
sudo apt-get update
sudo apt install g++ gdb make ninja-build rsync zip -y
sudo apt install bluetooth bluez libbluetooth-dev -y
```

## Command Line Options
 * -h (--help) Prints supported options and exits.
 * -l (--log) Sets the log directory
 * -t (--time) Sets the frequency data is written to the logs
 * -v (--verbose) Sets output verbosity.
 * -m (--mrtg) Takes a bluetooth address as parameter, returns data for that particular address in the format MRTG can interpret.
 * -o (--only) Takes a bluetooth address as parameter and only reports on that address.
 * -a (--average) Affects MRTG output. The parameter is a number of minutes. 0 simply returns the last value in the log file. Any number more than zero will average the entries over that number of minutes. If no entries were logged in that time period, no results are returned. MRTG graphing is then determined by the setting of the unknaszero option in the MRTG.conf file.
 * -d (--download) download the 20 days historical data from each device. This is still very much a work in progress.
 * -s (--svg) SVG output directory. Writes four SVG files per device to this directory every 5 minutes that can be used in standard web page. 
 * -c (--celsius) SVG output using degrees C
 * -b (--battery) Draw the battery status on SVG graphs. 1:daily, 2:weekly, 4:monthly, 8:yearly
 * -x (--minmax) Draw the minimum and maximum temperature and humidity status on SVG graphs. 1:daily, 2:weekly, 4:monthly, 8:yearly

 ## Log File Format

 The log file format has been stabe for a long time as a simple tab-separated text file with a set number of columns: Date, Temperature, Humidity, Battery.

 With the addition of support for the meat thermometers multiple temperature readings, I've changed the format slightly in a way that should be backwards compatible with most programs reading existing logs. After the existing columns of Date, Temperature, Humidity, Battery I've added optional columns of Model, Temperature, Temperature, Temperature

 ## Bluetooth UUID details
  * (UUID) 88EC (Name) Govee_H5074_C7A1
  * (Name) GVH5075_AE36 (UUID) 88EC
  * (Name) GVH5177_3B10 (UUID) 88EC
  * (UUID) 5182
  * (UUID) 5183

The 5074, 5075, and 5177 units all broadcast a UUID of 88EC. Unfortunately, the 5074 does not include the UUID in the same advertisment as the temperatures. 

The 5182 and 5183 units broadcast UUID of 5182 and 5183 respectivly in each of their broadcast messages including the temperatures.
```
(Flags) 06 (UUID) 5182 (Manu) 3013270100010164018007D0FFFF860708FFFF (Temp) 20�C (Temp) -0.01�C (Temp) 18�C (Temp) -0.01�C (Battery) 0%
(UUID) 5183 (Flags) 05 (Manu) 5DA1B401000101E40186076C2F660000 (Temp) 19�C (Temp) 121.34�C (Battery) 0% (Other: 00)  (Other: 00)  (Other: 00)  (Other: 00)  (Other: 00)  (Other: CB)
```

## BTData directory contains a Data Dump
The file btsnoop_hci.log is a Bluetooth hci snoop log from a Google Nexus 7 device running Android and the Govee Home App. It can be loaded directly in Wireshark.
 
In frames 260, 261, 313, 320, 11126, 11402, and 11403 you can see advertisements from my H5074 device. (e3:5e:cc:21:5c:0f)

Using the Govee Home App, I add a connection to Govee_H5074_5C0F and download it's historical data.

In frames 5718, 5719, 5728, and 11450 you can see advertisements from one of my H5075 devices. (a4:c1:38:37:bc:ae)

Interesting frames start around 543 in response to [UUID: 494e54454c4c495f524f434b535f2013].

Two sequential values are:
 * 708003611c0365010365000368ea0368ea0368ec
 * 707a0368eb036cd3036cd1036cd3036cd3036cd0

Looking at the data I believe that the first two bytes are an offset into the total data, and then there are six repeating three byte datasets. 

7080 03611c 036501 036500 0368ea 0368ea 0368ec

Using the same math that decodes the BT LE Advertisements gets very reasonable values. (71.86424, 46.8) (72.0437, 46.5) (72.04352, 46.4) (72.22388, 46.6) (72.22388, 46.6) (72.22424, 46.8)

If I zoom all the way to frame 5414, it appears to bt the last response to [UUID: 494e54454c4c495f524f434b535f2013] and has a value of 

0002 030c6b 030c70 ffffffffffffffffffffffff

Using the Govee Home App, I add a connection to GVH5075_BCAE and download it's historical data. 

The frames received from the thermometer start to look especially interesting around 5924 when they are returning consistent length data (20 bytes) similar to 7080031322031325031324031325031326031328 in response to [UUID: 494e54454c4c495f524f434b535f2013]
 * 7080031322031325031324031325031326031328
 * 707a031327031329031329031329031329031329

7080 031322 031325 031324 031325 031326 031328 

The last frame from [UUID: 494e54454c4c495f524f434b535f2013] (16687) has Value: 000603658f036590036590036590036590036590

0006 03658f 036590 036590 036590 036590 036590

Off the top of my head each device is storing 0x7080 time/humidity values. That's 28,800. 20 days * 24 hours * 60 minutes = 28,800 entries.

