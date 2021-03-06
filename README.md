# Python-mirobo

This project was started to allow controlling locally available Xiaomi Vacuum cleaner robot with Python (hence the name),
however, thanks to contributors it has been extended to allow controlling other Xiaomi devices using the same protocol.

Thanks for the nice people over [ioBroker forum](http://forum.iobroker.net/viewtopic.php?f=23&t=4898) who figured out the encryption to make this possible.
[Information about the  underlying communication protocol](https://github.com/OpenMiHome/mihome-binary-protocol) ([another source for vacuum-specific documentation](https://github.com/marcelrv/XiaomiRobotVacuumProtocol))

## Supported devices

* Xiaomi Mi Robot Vacuum
* Xiaomi Mi Air Purifier 2
* Xiaomi Mi Smart WiFi Socket
* Xiaomi Mi Smart Socket Plug (1 Socket, 1 USB Port)
* Xiaomi Smart Power Strip (WiFi, 6 Ports)
* Xiaomi Philips Eyecare Smart Lamp 2
* Xiaomi Philips LED Ceiling Lamp
* Xiaomi Philips LED Ball Lamp
* Xiaomi Universal IR Remote Controller (Chuangmi IR)
* Xiaomi Mi Smart Fan
* Yeelight light bulbs (only a very rudimentary support, use [python-yeelight](https://gitlab.com/stavros/python-yeelight/) and for a more complete support)

*Feel free to crete pull requests to add support for new devices as well as to report missing functionalities in the currently supported devices.*

## Adding support for other devices

The [miio javascript library](https://github.com/aholstenson/miio) contains some hints on devices which could be supported, however, the Xiaomi Smart Home gateway ([Home Assistant component](https://github.com/lazcad/homeassistant) already work in progress) as well as Yeelight bulbs are currently not in the scope of this proejct.

# Installation

Please make sure you have libffi and openssl headers installed, you can do this on Debian-based systems (like Rasperry Pi) with ```apt-get install libffi-dev libssl-dev```.
Also do note that the setuptools version is too old for installing some requirements, so before trying to install this package you should update the setuptools with ```pip3 install -U setuptools```.

The easiest way to install the package is to use pip: ```pip3 install python-mirobo``` . [Using virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/) is recommended.

In case you get an error similar like ```ImportError: No module named 'packaging'``` during the installation,
you need to upgrade pip and setuptools:

```
$ pip3 install -U pip setuptools
```

# Getting started

As long as the device is in the same network, `mirobo discover` can be used to check for its support status.

To be able to communicate with supported devices its IP address and an encryption token must be known.
The token can be obtained either by extracting it from the database of the Mi Home application,
or by using the automatic discovery.

## Finding the token

*Important* For the Mi Robot Vacuum Cleaner with firmware 3.3.9_003077 or higher follow these steps to get the token: https://github.com/jghaanstra/com.xiaomi-miio/blob/master/docs/obtain_token_mirobot_new.md ([another source](https://github.com/homeassistantchina/custom_components/blob/master/doc/chuang_mi_ir_remote.md)).

This will also work for all other devices as long as the device has been bound with the Mi Home mobile application, and is therefore preferable way to attain the token.

### Automatic discovery (does not work on all devices and firmware versions)

The automatic discovery works only before the device has been connected over the app to your local wifi.
In order to fetch the token, reset the device, connect to its the network its announcing (e.g. rockrobo-XXXX for the vacuum)
and run the following command:

```
mirobo discover --handshake 1
```

which should return something similar to this:

```
INFO:mirobo.vacuum:  IP 192.168.8.1: Xiaomi Mi Robot Vacuum - token: b'ffffffffffffffffffffffffffffffff'
```

If the value is as shown above, the vacuum has already been connected and it needs a reset.
Otherwise the token can be copied over and used for controlling.


# Controlling the vacuum cleaner

Following features of the vacuum cleaner are currently supported:

* Basic functionality, including starting, stopping, pausing, locating.
* Controlling the fan speed.
* Fetching schedule, status, state of consumables. *Reseting consumable state and setting schedules is not currently implemented, patches welcome!*
* Setting and querying the timezone.
* Manual control of the robot.

Use `mirobo --help` for more information about supported commands.


## Usage

To simplify the use, instead of passing the IP and the token as a parameter for the tool,
you can simply set the following environment variables.

```
export MIROBO_IP=192.168.1.2
export MIROBO_TOKEN=476e6b70343055483230644c53707a12
```

After that verify that the connection is working by running the command withour parameters,
you should be presented a status report from the vacuum.

Use ```mirobo --help``` to see available commands and description about what they do.
```--debug``` option can be used to let it show raw JSON data being communicated.

### DND functionality

To disable:
```
mirobo dnd off
```

To enable (dnd 22:00-0600):
```
mirobo dnd on 22 0 6 0
```

It is also possible to run raw commands for testing:

```
mirobo raw_command app_start
```

or with parameters (same as above dnd on):
```
mirobo raw_command set_dnd_timer '[22,0,6,0]'
```

If you find a new command please let us know by creating a pull request or an issue, if you
do not want to implement it yourself!

## Usage examples

### Status reporting

```
$ mirobo
State: Charging
Battery: 100
Fanspeed: 60
Cleaning since: 0:00:00
Cleaned area: 0.0 m²
DND enabled: 0
Map present: 1
in_cleaning: 0
```

### Start cleaning

```
$ mirobo start
Starting cleaning: 0
```

### Return home

```
$ mirobo home
Requesting return to home: 0
```

### Setting the fanspeed

```
$ mirobo fanspeed 30
Setting fan speed to 30
```

### State of consumables

```
$ mirobo consumables
main: 9:24:48, side: 9:24:48, filter: 9:24:48, sensor dirty: 1:27:12
```

### Schedule information

```
$ mirobo timer
Timer #0, id 1488667794112 (ts: 2017-03-04 23:49:54.111999)
  49 22 * * 6
  At 14:49 every Saturday
Timer #1, id 1488667777661 (ts: 2017-03-04 23:49:37.661000)
  49 21 * * 3,4,5,6
  At 13:49 every Wednesday, Thursday, Friday and Saturday
Timer #2, id 1488667756246 (ts: 2017-03-04 23:49:16.246000)
  49 20 * * 0,1,2
  At 12:49 every Sunday, Monday and Tuesday
Timer #3, id 1488667742238 (ts: 2017-03-04 23:49:02.237999)
  49 19 * * 0,6
  At 11:49 every Sunday and Saturday
Timer #4, id 1488667726378 (ts: 2017-03-04 23:48:46.378000)
  48 18 * * 1,2,3,4,5
  At 10:48 every Monday, Tuesday, Wednesday, Thursday and Friday
Timer #5, id 1488667715725 (ts: 2017-03-04 23:48:35.724999)
  48 17 * * 0,1,2,3,4,5,6
  At 09:48 every Sunday, Monday, Tuesday, Wednesday, Thursday, Friday and Saturday
Timer #6, id 1488667697356 (ts: 2017-03-04 23:48:17.355999)
  48 16 5 3 *
  At 08:48 on the 5th of March
```

### Cleaning history

```
$ mirobo cleaning_history
Total clean count: 43
Clean #0: 2017-03-05 19:09:40-2017-03-05 19:09:50 (complete: False, unknown: 0)
  Area cleaned: 0.0 m²
  Duration: (0:00:00)
Clean #1: 2017-03-05 16:17:52-2017-03-05 17:14:59 (complete: False, unknown: 0)
  Area cleaned: 32.16 m²
  Duration: (0:23:54)

```


## Home Assistant support

See [Xiaomi Mi Robot Vacuum](https://home-assistant.io/components/vacuum.xiaomi/).
