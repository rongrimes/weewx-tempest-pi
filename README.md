# weewx-tempest-pi

#### Development Note
This document is under development. The content is advanced enough to enable you to set up *weewx* to retrieve data from your Weatherflow Tempest from a Raspberry Pi. Formatting, layout and content clarification are under active development.

Feel free to let me know if you're interested in the progress of this document.

#### Another good source of information

Much of my content is derived from:  
* https://github.com/captain-coredump/weatherflow-udp

However, there was not a straight forward cookbook approach to setting up *weewx* with a Weatherflow Tempest; hence this document. I hope it helps someone.

May 2022

---

## Introduction

This document describes setting up *weewx* to process the results from a **Weather Flow Tempest**.

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
I have had the _Weatherflow Tempest_ running with _weewx_ software on a _Raspberry Pi 4 2 GB_ platform for 9 months. I think that a better $ value is to have _weewx_ running on a _Pi Zero 2_. 

This document is being written as I convert the station from the _Pi 4_ to _Pi Zero 2_. Since I am converting from an existing system, I may leave a few bits out. If they are annoying, or the document could be improved, please let me know. If otherwise, then, just: Sorry!

---

## Raspberry PiOS

I am using:<br/>
![Raspberrpi Lite (64-bit)](RPi64Lite.png)

### Installation
- Install onto an SD card as usual (I use _rpi_imager_)
- Install SD card into the Pi Zero and start up as usual.
- Configure _ssh_ to run for remote access.

#### Other software load
I use `vim` for editing, and the text below uses vim. Any other editor is just as suitable. Here is the installation command for reference:
```
sudo apt install vim
```

### dietpi
I tried to install _weewx_ with the [**dietpi**](https://dietpi.com/) distro. _weewx_ found all sorts of modules missing and it just wasn't worth the effort to continue. 

---

## weewx

The guide at [**WeeWX: Installation on Debian-based systems**](https://weewx.com/docs/debian.htm) describes installing *weewx* on Debian / Reaspberry Pi systems. The steps below give you a guide through the essential sections.

### Retrieve, Install weewx

On the Weewx Installation page, follow the topics:
- [Configure apt](https://weewx.com/docs/debian.htm#configure_apt) - shows specifics of retrieving with *apt*
- [Install](https://weewx.com/docs/debian.htm#Install) *weewx*

#### Installation Notes

The _weewx_ installation will ask for the following:

| Value | Note |
| --- | --- |
| **Location of Weather Station** | Enter the name/location of the station. Use the Tempest name, but any name will do (example: Cherry Beach) |
| **latitude, Longitude** | Enter the decimal values of the site co-ordinates.<br/>(Note: installation allows any input here and doesn't check. However, hidden system errors occur at runtime when you supply badly formed values.) |
| **Altitude** | Enter site altitude as directed. |
| **display units** | Make a choice. Further adjustments are easy to make later in the configuration file. |
| **Weather Station hardware Type** | Choose: **Simulator**. |

### Status

On the Weewx Installation page, follow the topics:

| Value | Note |
| --- | --- |
| **Status** | Log entries will appear. These values are unimportant at this time. |
| **Verify** | Web pages to appear at `/var/www/html/weewx/index.html` |
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

### Replace [[sensor_map]]

- Discard [[sensor_map]] contents.
- Follow https://github.com/captain-coredump/weatherflow-udp/blob/master/sample_Tempest_sensor_map
- Insert [[sensor_map]] contents from `sample_Tempest_sensor_map`, or from below:
```
 # This section is for the TEMPEST WeatherFlow Bridge packets, via UDP broadcast on local subnet

    [[sensor_map]]
        outTemp = air_temperature.ST-00000025.obs_st
        outHumidity = relative_humidity.ST-00000025.obs_st
        pressure = station_pressure.ST-00000025.obs_st
        #lightning_strikes =  lightning_strike_count.ST-00000025.obs_st
        #avg_distance =  lightning_strike_avg_distance.ST-00000025.obs_st
        outTempBatteryStatus = battery.ST-00000025.obs_st
        windSpeed = wind_speed.ST-00000025.rapid_wind
        windDir = wind_direction.ST-00000025.rapid_wind
        #luxXXX = illuminance.ST-00000025.obs_st
        UV = uv.ST-00000025.obs_st
        rain = rain_accumulated.ST-00000025.obs_st
        windBatteryStatus = battery.ST-00000025.obs_st
        radiation = solar_radiation.ST-00000025.obs_st
        #lightningXXX = distance.ST-00000025.evt_strike
        #lightningYYY = energy.ST-00000025.evt_strike
```


### Get Your Station Identification
The sample code is for data coming from station ID `ST-00000025`. You now need to find out *your* station ID.
  
(following captain-coredump/weatherflow-udp):
  
- Set `log_raw_packets = True`
- Restart weewx.  
  `sudo /etc/init.d/weewx restart`
  
*weewx* will start watching for the UDP packets from the Tempest and dump them in the log. We can see this information with:
  
 `sudo tail -f /var/log/syslog`
  
 You are looking for records like:
  ```
 May 26 22:26:55 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'rapid_wind', 'hub_sn': 'HB-00041000', 'ob': [1653618412, 0.52, 315]}
May 26 22:28:25 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'rapid_wind', 'hub_sn': 'HB-00041000', 'ob': [1653618504, 0.29, 312]}
May 26 22:28:26 raspberrypiZ2-2 weewxd: weatherflowudp: MainThread: raw packet: {'serial_number': 'ST-00052000', 'type': 'device_status', 'hub_sn': 'HB-00041000', 'timestamp': 1653618505, 'uptime': 26712917, 'voltage': 2.763, 'firmware_revision': 156, 'rssi': -64, 'hub_rssi': -64, 'sensor_status': 131072, 'debug': 0}
  ```
 (I have slightly obfuscated the serial numbers from my own Weatherflow Tempest).
  
 Terminate the log viewing with Ctl-C.
  
 ### Insert your Serial number into weewx.conf
  
 - We want the Tempest serial number (here: ST-000520000) in the sensor map code:  
  In /etc/weewx/weewx.conf, search/replace ST-00000025, and replace with ST-00052000 (but with what you found in your log file)
 - Restart weewx.  
  `sudo /etc/init.d/weewx restart`
 - Let run for 10 - 15 minutes (or more).
  
 ### View web pages
*weewx* has main ouput in web pages at: `/var/www/html/weewx`. To see if *weewx* is working for you, view the index.html file.  
`vim /var/www/html/weewx/index.html`

Browse down and look for output like:
```
  <div class="widget_contents">
  <table>
    <tbody>
      <tr>
        <td class="label">Outside Temperature</td>
        <td class="data">61.7&#176;F</td>
      </tr>
      <tr>
        <td class="label">Heat Index</td>
        <td class="data">61.0&#176;F</td>
      </tr>
      <tr>
        <td class="label">Wind Chill</td>
        <td class="data">61.7&#176;F</td>
      </tr>
      <tr>
        <td class="label">Dew Point</td>
        <td class="data">52.6&#176;F</td>
      </tr>
```
What's notable here is:  
**Outside Temperature** appears with a value of **61.7**. If *weewx* is not configured correctly, you will likely see **N/A**.

---

## Further Configuration

### Measurement Units

The default measurement units for *StdReport* appear in `/etc/weewx/weewx.conf` and are set to US units. I set mine to the following (Canadian standard).

1. In `/etc/weewx/weewx.conf`, navigate to: **StdReport** >> **Defaults**
1. Set `unit_system = metric`
1. Navigate a few lines down to **Units** >> **Groups**, and edit the respective lines to read:
```
    group_pressure  = kPa          # Options are 'inHg', 'mmHg', 'mbar', 'hPa', or 'kPa'
    group_rain      = mm           # Options are 'inch', 'cm', or 'mm'
    group_rainrate  = mm_per_hour  # Options are 'inch_per_hour', 'cm_per_hour', or 'mm_per_hour'
```
4. Restart *weewx*.
```
sudo /etc/init.d/weewx restart
```
---

### Output to web site using FTP

I use <a href="https://www.infinityfree.net/" target="_blank">Infinity Free</a> as a web hosting site. It's:
* free
* allows regular ftp upload
* is mostly smooth in producing pages

... and suits my non-professional purposes.

Use the notes in [FTP](https://weewx.com/docs/usersguide.htm#config_FTP) in the weewx users guide for FTP transfer.

For illustration, I have configured:
```
enable = true
user = epiz_redacted
password = redacted
server = ftpupload.net    # The ftp server name, e.g, www.myserver.org
path = /rongrimes.42web.io/htdocs    # The destination directory, e.g., /weather

# Set to True for an FTP over TLS (FTPS) connection. Not all servers
# support this.
secure_ftp = False

# Most FTP servers use port 21
port = 21

# Set to 1 to use passive mode, zero for active mode
passive = 1
```

#### Edit template(s)

---

## Additional reporting sites

### Weather Underground

### AWEKAS

### Weather Cloud, etc

---

## Implementation Notes
### Network Connection: Wireless > Ethernet

I found the wireless connection got faulty after a few days. The symptom being that *weewx* would lose data and show temperature (and other readings) as **N/A** with  the daily graphs being dots instead of continuous lines.

Cure: I turned off wireless and used ethernet-to-usb. The data reception was rock solid from then on.

---

---

## My Scratch area
This is a "pending notes" area. These notes will eventually be added into the main body, or discarded.

#### TODO
1. install weatherunderground, AWEKAS, etc.
2. Add references.
