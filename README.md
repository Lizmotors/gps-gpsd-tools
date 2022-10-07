# gps-gpsd-tools
Configure GPS Reciever on ubuntu

    GPS Receiver:-
    I would go for a generic USB GPS receiver such as the BU-353 which is plug & play
    device. We need hang this device in open place so that it will receive enough signal

Steps to configure GPS device:-        

Install GPS Packages and Configure

          $ sudo apt-get update

          $ sudo apt-get upgrade

          $ sudo rpi-update

          $ sudo apt-get install gpsd gpsd-clients python-gps ntp

          $ sudo shutdown -r now

    $ lsusb

            Bus 001 Device 004: ID 067b:2303 Prolific Technology, Inc. LP2303 Serial Port

    Edit “/etc/default/gpsd”
    # Default settings for the gpsd init script and the hotplug wrapper.
    # Start the gpsd daemon automatically at boot time
    START_DAEMON=”true”
    # Use USB hotplugging to add new USB devices automatically to the daemon
    USBAUTO=”true”
    # Devices gpsd should collect to at boot time.
    # They need to be read/writeable, either by user gpsd or the group
    dialout.
    DEVICES=”/dev/ttyUSB0″
    # Other options you want to pass to gpsd
    GPSD_OPTIONS=”-F /var/run/gpsd.sock -b -n”

    Restart the gpsd Service                                                                                                                          

$ sudo systemctl enable gpsd
$ sudo systemctl start gpsd
$ sudo systemctl status gpsd
gpsd.service – GPS (Global Positioning System) Daemon
Loaded: loaded (/lib/systemd/system/gpsd.service; static)
Active: active (running) since 水 2015-09-30 19:59:24 JST; 2min 14s ago
Main PID: 1089 (gpsd)
CGroup: /system.slice/gpsd.service
mq1089 /usr/sbin/gpsd -N -F /var/run/gpsd.sock -b -n /dev/ttyUSB0

    cgps -s

           This will display the details of your current location with values.

    Edit “/etc/ntp.conf” add Lines under “server 3.debian.pool.ntp.org iburst   

# gps ntp

server 127.127.28.0 minpoll 4
fudge 127.127.28.0  time1 0.183 refid NMEA
server 127.127.28.1 minpoll 4 prefer
fudge 127.127.28.1  time1 0.183 refid PPS

    To allow the network to query your NTP server. Add your Network like:
    restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

    Restart the ntp service

           $ sudo systemctl enable ntp
$ sudo systemctl start ntp
$ sudo systemctl status ntp
ntp.service – LSB: Start NTP daemon
Loaded: loaded (/etc/init.d/ntp)
Active: active (running) since 水 2015-09-30 19:59:31 JST; 14min ago
CGroup: /system.slice/ntp.service
mq1117 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 107:112

     $ ntpq -p

             It should display time from  SHM(0) & SHM(1)
             
##########################################################################################
GPSD Tools

Link:
https://gpsd.gitlab.io/gpsd/index.html

$cgps

$gpsmon
$xgps

$ gpspipe -r
$gpsprof | gnuplot -persist

#####################################
GPS Data logger 

gpspipe -r

Automatically Capture Data on Boot.

We will be using gpspipe to capture the NMEA sentence from the BerryGPS and storing these into a file. The command to use is;

gpspipe -r -d -l -o /home/usr/`date +”%Y%m%d-%H-%M-%S”`.nmea

-r = Output raw NMEA sentences.
-d = Causes gpspipe to run as a daemon.
-l = Causes gpspipe to sleep for ten seconds before attempting to connect to gpsd.
-o = Output to file.

The date the file is created is also added to the name.

Now we need to force the above command to run at boot. This can be done by editing the rc.local file.

$ sudo nano /etc/rc.local

ust before the last line, which will be ‘exit 0’, paste in the below line;

gpspipe -r -d -l -o /home/pi/`date +"%Y%m%d-%H-%M-%S"`.nmea

Reboot and confirm that you can see a .nmea file in the home directory.

Every time the computer is rebooted, a new file will be created.
