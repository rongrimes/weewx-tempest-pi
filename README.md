# weewx-pi

#### Development
This note is under development. I am updating as I get the installation correct.<br/>
DO NOT USE yet.<br/>

However, feel free to let me know if you're interested in the progress of this document.

#### Another good source of information

* https://github.com/captain-coredump/weatherflow-udp

May 2022

---

## Introduction

This note describes setting up a **Raspberry Pi** to process the results from a **Weather Flow Tempest**.

I have the following hardware:

| Weatherflow Tempest | Raspberry Pi Zero 2 W |
|---|---|
| <img src="https://cdn.shopify.com/s/files/1/0012/8512/8294/products/Tempest_Hub_Mount_shopify-amazon-2020_1024x1024@2x.png" width="200"> | <img src="https://assets.raspberrypi.com/static/51035ec4c2f8f630b3d26c32e90c93f1/2b8d7/zero2-hero.webp" width="200"> |

... with a goal to integrate and broadcast over public weather networks:
* **Weatherflow Tempest Reporting**
* **Weather Underground**
* **AWEKAS**
* **Weather Cloud**
* **weewx Weather reporting**
* **WOW**

### Full disclosure :)  
I have had the _Tempest_ running with _weewx_ software on a _Raspberry Pi 4 2 GB_ platform for 9 months. I think that a better $ value is to have _weewx_ running on a _Pi Zero 2_. 

This document is being written as I convert the station from the _Pi 4_ to _Pi Zero 2_. Since I am converting from an existing system, I may leave a few bits out. If they are annoying, or the document could be improved, please let me know. If otherwise, then, just Sorry!

---

## Raspberry PiOS

I am using:<br/>
![Raspberrpi Lite (64-bit)](RPi64Lite.png)

### Installation
- Install onto an SD card as usual (I use _rpi_imager_)
- Install SD card into the Pi Zero and start up as usual.
- Configure _ssh_ to run for remote access.

### dietpi
I tried to install _weewx_ with the [**dietpi**](https://dietpi.com/) distro. _weewx_ found all sorts of modules missing and it just wasn't worth the effort to continue. 

---

## weewx

To retrieve and install weewx, follow the guide at [**WeeWX: Installation on Debian-based systems**](https://weewx.com/docs/debian.htm)

### Retrieve, Install weewx

On the Weewx Installation page, follow the topics:
- [Configure apt](https://weewx.com/docs/debian.htm#configure_apt)
- [Install](https://weewx.com/docs/debian.htm#Install)

#### Installation Notes

The _weewx_ installation will ask for the following:

| Value | Note |
| --- | --- |
| **Location of Weather Station** | Enter the name/location of the station. Use the Tempest name, but any name will do (example: Cherry Beach) |
| **latitude, Longitude** | Enter the decimal values of the site co-ordinates.<br/>(Note: installation allows any input here and doesn't check. However, hidden system errors occur at runtime with badly formed values.) |
| **Altitude** | Enter site altitude as directed. |
| **display units** | Make a choice. Further adjustments are easy to make later in the configuration file. |
| **Weather Station hardware Type** | Choose: **Simulator**. |

### Status

On the Weewx Installation page, follow the topics:

| Value | Note |
| --- | --- |
| **Status** | Log entries will appear. There values are unimportant at this time. |
| **Verify** | Web pages should appear at `/var/www/html/weewx/index.html` |
| **Customize** | Ignore (for now). |
| **Start/Stop** | Stop _weewx_. (`sudo /etc/init.d/weewx stop`) |
| **Uninstall** | Ignore. |
| **Layout** | Keep for reference. |

--- 
## Install Weather Flow Tempest module

### Retrieve UDP code
- Visit https://github.com/captain-coredump/weatherflow-udp
- Download the .ZIP download of the project from the GitHub web interface
  - Button: `CODE`
  - Choose `Download ZIP`

This retrieves: `weatherflow-udp-master.zip`

### Install
- `sudo wee_extension --install weatherflow-udp-master.zip`

### Edit weewx.config

First, stop weewx.
  `sudo /etc/init.d/weewx stop`


```
cd /etc/weewx
sudo vim weewx.conf
```

- Find the section `[Simulator]`, and delete / comment out all lines in the section.
- Note: The sections: `[Station] ` (above), and `[StdRESTful]` remain intact.
- Copy the following section to where `[Simulator]` was previously:

```
[WeatherFlowUDP]
    driver = user.weatherflowudp
    log_raw_packets = False
    udp_address = <broadcast>
    # udp_address = 0.0.0.0
    # udp_address = 255.255.255.255
    udp_port = 50222
    udp_timeout = 90
    share_socket = False

    [[sensor_map]]
        outTemp = air_temperature.AR-00004444.obs_air
        outHumidity = relative_humidity.AR-00004444.obs_air
        pressure =  station_pressure.AR-00004444.obs_air
        # lightning_strikes =  lightning_strike_count.AR-00004444.obs_air
        # avg_distance =  lightning_strike_avg_distance.AR-00004444.obs_air
        outTempBatteryStatus =  battery.AR-00004444.obs_air
        windSpeed = wind_speed.SK-00001234.rapid_wind
        windDir = wind_direction.SK-00001234.rapid_wind
        # lux = illuminance.SK-00001234.obs_sky
        UV = uv.SK-00001234.obs_sky
        rain = rain_accumulated.SK-00001234.obs_sky
        windBatteryStatus = battery.SK-00001234.obs_sky
        radiation = solar_radiation.SK-00001234.obs_sky
        # lightningYYY = distance.AR-00004444.evt_strike
        # lightningZZZ = energy.AR-00004444.evt_strike

```
(original source: https://github.com/captain-coredump/weatherflow-udp)

I edited the top part to be:
```
[WeatherFlowUDP]
    driver = user.weatherflowudp
    log_raw_packets = False
    #   log_raw_packets = True
    #   udp_address = <broadcast>
    #   udp_address = 0.0.0.0
    udp_address = 255.255.255.255
    udp_port = 50222
    udp_timeout = 90
    share_socket = False
```
  
In the top section (`[Station]`), edit `station_type` to:  
`station_type = WeatherFlowUDP`

### Station Identification
The sample code is for data coming from station ID `AR-00004444`. You now need to find out *your* station ID.
  
(following captain-coredump/weatherflow-udp):
  
- Set `log_raw_packets = True`
- Restart weewx.  
  `sudo /etc/init.d/weewx start`
  
*weewx* will start watching for the UDP packets from the Tempest and dump them in the log. We can see this information with:
  
 `sudo tail -f /var/log/syslog`
  
 You are looking for records like:
  ```
 May 26 22:26:55 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'rapid_wind', 'hub_sn': 'HB-00041000', 'ob': [1653618412, 0.52, 315]}
May 26 22:28:25 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'rapid_wind', 'hub_sn': 'HB-00041000', 'ob': [1653618504, 0.29, 312]}
May 26 22:28:26 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'device_status', 'hub_sn': 'HB-00041000', 'timestamp': 1653618505, 'uptime': 26712917, 'voltage': 2.763, 'firmware_revision': 156, 'rssi': -64, 'hub_rssi': -64, 'sensor_status': 131072, 'debug': 0}
  ```
 (I have slightly obfuscated the serial numbers).
  
 Terminate the log viewing with Ctl-C.
  
 ### Insert your Serial number into weewx.conf
  
 - Stop *weewx* (`sudo /etc/init.d/weewx stop`)
 - We want the Tempest serial number (here: ST-000520000) in the sensor map code:  
  In /etc/weewx/weewx.conf, search/replace AR-00004444 (and SK-00001234), and replace with ST-000052000 (but with what you found in your log file)
 - Restart weewx.  
  `sudo /etc/init.d/weewx start`
 - Let run for 10 - 15 minutes (or more).
  
 ### View web pages


---

## Further Configuration
#### Weather Underground

### Output to web site

---

## Edit template(s)

---

## Additional reporting sites

### AWEKAS

### Weather Cloud, etc
