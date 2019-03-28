# How-To: Xiaomi Vacuum robot Gen 1 root update #

Big thanks to @dgiese, @JackGruber, @Hypfer and everybody who contributed. 

## Introducton ##
This guide provides information how to root Xiaomi Vacuum robot, install software on the robot and also how to connect robot to Home Assistant (see part below)

This guide is made on top of existing guides (see list below). These guides are a bit confusing, that is the reason why this guide was created.

- Dustcloud: https://github.com/dgiese/dustcloud
- How-to dustcloud guide (short-version, without Valetudo): https://github.com/dgiese/dustcloud/wiki/VacuumRobots-manual-update-root-Howto
- How-to dustcloud guide (long-version, with Valetudo) - this version is used below and enriched with my comments: https://github.com/dgiese/dustcloud/wiki/Cloud-Free-Firmware-Image-With-Valetudo
- Dustcloud on Docker: https://github.com/JackGruber/docker_dustcloud
- List of firmware mirror: https://github.com/dgiese/dustcloud/tree/master/devices/xiaomi.vacuum.gen1/firmware (currently down, last checked 20190318)

## Explanation of used components ##
- Dummycloud - Network settings on vacuum that is installed with rooted firmware. Simply blocks communication with Xiaomi servers.
- Dustcloud (https://github.com/dgiese/dustcloud) - Aapplication which serves as vacuum server. Needs to run on separate machine in your network. It is not required when you are using Valetudo or Valetudo + Home Assistant
- Valetudo (https://github.com/Hypfer/Valetudo) - Vacuum UI that runs directly on the vacuum. Can be installed with rooted firmware.
- Home Assistant (https://www.home-assistant.io/) - Home automation platform. 

## What you need ##
- Linux system, preferably virtual instance. Tested on Debian 9
- Xiaomi Vacuum robot Gen 1 (should also work on Gen 2)
- If you want connection to Home Assistant, you need to have it installed somewhere - I used Raspberry Pi 3

## Steps ##
There are two main setups how this procedure can be made: 
- Rooted robot + Dustcloud installed (dustcloud can run in Docker - https://github.com/JackGruber/docker_dustcloud)
- Rooted robot with Valetudo + optional connection to Home Assistant
This guide mainly focuses on the second approach, but I have also tried the first


__These steps are copied from https://github.com/dgiese/dustcloud/wiki/Cloud-Free-Firmware-Image-With-Valetudo__

_You can find my notes in following sections as bold_

## Suggested firmware versions

* **Gen1:** v11_003468
* **Gen2:** v11_001738

__Note: For Gen1 I used v11_003600__

## Intro

You will want to use a Linux based system. It doesn't matter if it is your base OS on your PC/Laptop, a VM on your PC, a Server or any other device like a Raspi. Chose the Distro you like (I'm more the Debian guy).
Make sure it has access to your LAN/Internet. It does NOT NEED WIFI!

__Note: I have used Debian 9 Linux running on VMWare. This virtual machine is set to Bridged networking to my Wi-Fi__

## Building the firmware

### Install required dependencides

You should make sure that the following packages are installed in your system:

* bash
* openssh (for ssh-keygen)
* ccrypt
* sed
* dos2unix

### Prerequisites

You need to have a public / private ssh-key pair to connect to the robot. If you do not have a keypair yet, you can generate one with the following command
```
ssh-keygen -t ed25519 -C "your_email@example.com"
```
Per default, the generated keys will be created under `~/.ssh`. 
If you choose to create the keys in another location, remember your chosen location for later.

__Note:This key is later used as authentication method for SSH running on the Vacuum__

### Preparations for building the image

1. Create a new directory for your work

       mkdir rockrobo
       cd rockrobo
    
2. Clone the dustcloud repository (until imagebuilder > 0.1 is available)
       
       git clone https://github.com/dgiese/dustcloud.git
       
4. Create a directory for dummycloud

       mkdir dummycloud
       pushd dummycloud
       
5. Download dummycloud from https://github.com/dgiese/dustcloud/releases

       wget https://github.com/dgiese/dustcloud/releases/download/0.1/dummycloud_0.1.zip
       unzip -a dummycloud*.zip
       popd
       
6. Create a valetudo directory

       mkdir valetudo
       pushd valetudo
      
7. Download the latest valetudo binary from https://github.com/Hypfer/Valetudo/releases

       wget https://github ...
       mkdir deployment
       pushd deployment
       wget https://github.com/Hypfer/Valetudo/raw/master/deployment/valetudo.conf
       popd
       popd

8. Create rrlogd-patcher directory

       mkdir rrlogd-patcher
       pushd rrlogd-patcher
       
9. Download the latest `patcher.py`

       wget https://raw.githubusercontent.com/JohnRev/rrlogd-patcher/master/patcher.py
       popd

8. Create firmware directory

       mkdir firmware
       pushd firmware
       
9. Download the latest firmware (e.g. v001792)

       wget URL

**URL:**

**Gen1**

`https://cdn.awsbj0.fds.api.mi-img.com/updpkg/[package name]
https://cdn.awsde0.fds.api.mi-img.com/updpkg/[package name]
Example: https://cdn.awsbj0.fds.api.mi-img.com/updpkg/v11_003468.fullos.pkg`

**Gen2**

`https://cdn.awsbj0.fds.api.mi-img.com/rubys/updpkg/[package name]
https://cdn.cnbj2.fds.api.mi-img.com/rubys/updpkg/[package name]
https://cdn.cnbj0.fds.api.mi-img.com/rubys/updpkg/[package name]
https://cdn.awsde0.fds.api.mi-img.com/rubys/updpkg/[package name]
Example: https://cdn.awsbj0.fds.api.mi-img.com/rubys/updpkg/v11_001720.fullos.pkg`

__Note: server cdn.awsbj.fds.api-mi-img is official Xiaomi server. I have downloaded the firmware from there. For more information about firmware, check following page: https://github.com/dgiese/dustcloud/wiki/Xiaomi-Vacuum-Firmware For example I have used 3.3.9_003600	v11_003600.pkg for Gen 1__

10. Download sound package

        wget https://github.com/dgiese/dustcloud/raw/master/devices/xiaomi.vacuum/original-soundpackages/encrypted/english.pkg
        popd

If you followed the above commands, your `rockrobo` directory structure should now look like this:
```
tree -L 2 rockrobo/
rockrobo/
├── dummycloud
│   ├── doc
│   ├── dummycloud
│   ├── dummycloud_0.1.zip
│   └── README.md
├── dustcloud
│   ├── cloudprotocol.pdf
│   ├── devices
│   ├── docker
│   ├── dummycloud
│   ├── dustcloud
│   ├── LICENSE
│   ├── Pipfile
│   ├── Pipfile.lock
│   ├── presentations
│   └── README.md
├── firmware
│   ├── english.pkg
│   └── v11_001712.pkg
├── rrlogd-patcher
│   └── patcher.py
└── valetudo
    ├── deployment
    └── valetudo

```
Next, we can create the firmware image.

### Creating the firmware image

To create the firmware image you should run the following commands:

```
mkdir image
cd image     
sudo ../dustcloud/devices/xiaomi.vacuum/firmwarebuilder/imagebuilder.sh \
     --firmware=../firmware/v11_001712.pkg \
     --soundfile=../firmware/english.pkg \
     --public-key=$HOME/.ssh/id_ed25519.pub \
     --dummycloud-path=../dummycloud \
     --valetudo-path=../valetudo \
     --disable-firmware-updates \
     --ntpserver=fritz.box \
     --rrlogd-patcher=../rrlogd-patcher/patcher.py \
     --replace-adbd
```
__Note: Change package name (--firmware) and ssh key (--public-key) according to your setup. Use all parameters as shown__ 
__Note2: There might be a problem with UTF encoding. See following page for more informaton how to fix it: https://webkul.com/blog/setup-locale-python3/__

Here you need now to remember the location of your generated public ssh-key, `id_ed25519.pub`. If you don't have a public key yet, take a look at the Prerequisites section.

Note that not all options are required. However if your router runs a ntp server you should use it. Also I would recommend to replace adbd so in case something goes really wrong you can still access it via USB.

Make sure that rrlogd gets patched, (see Appendix for supported versions). You should see something like:

    Creating backup of rrlogd
    Trying to patch rrlogd
    Successfully patched rrlogd

**WARNING**: If you patch rrlogd you should use dummycloud or your data will be sent unencrypted to Xiaomi servers!

__Note: Ignore this warning. When you are using all parameters, you will automatically install dummy cloud on the Vacuum__

## Flashing the firmware image

After the successful build of the firmware image, we can tell the robot to download and flash it.
First, we need to create a virtual environment for it in python. For this the following packages need to be installed:

* python3
* python3-pip (https://linuxize.com/post/how-to-install-pip-on-debian-9/)
* python3-venv

```
cd ..
mkdir flasher
cd flasher
python3 -m venv venv
```

and install the required miio python packages:

```
source venv/bin/activate
pip3 install wheel
pip3 install python-miio
cd ..
```

__Note: I have found out, that more reliable way to flash the vacuum is to reset the Wi-Fi and use second method (see below). Also you do not need to look for vacuum token__

## First method ##
You do NOT NEED to reset your robo or change to a device with WIFI. The following command just requires you to know the internaal IP of your robot (get it from your router, you may even set the IP to never refresh) and the already existing token (use google for the russian Mi Home app - it was the quickest and easiest way I was able to achieve it):

__Note: Russian app for Android can be found here: http://www.kapiba.ru/2017/11/mi-home.html__

```
python3 dustcloud/devices/xiaomi.vacuum/firmwarebuilder/flasher.py -a X.X.X.X -t XXXXXXXXXXXXXXXX -f image/output/v11_001792.pkg
```

In this case you do not need to setup your WIFI on the robot again, like described at the bottom of this page.

After the successful transfer of the image to the robot, the robot will start flashing the image. This will take about 5~10 minutes. After the process is done, the robot will state that the update was successful.

Eventually, we can log in to our robot via ssh:

```
ssh root@192.168.8.1
```

If everything works out, you can now log in to your robot - enjoy.

You're done now using this method. You can proceed to: "Check if dummycloud is correctly set up"

If you face any trouble, check: "Flashing doesn't work"

## Second method ##

__Note: My vacuum has pretty bad wifi modem inside. Make sure that the vacuum and your computer is close as possible__

Next, turn your vacuum on and place it in the charger docking station. Connect yourself via WLAN to your robot, the SSID should be sth. like 

`roborock-vacuum-s5_miap_****`

Now we are finally ready to flash the firmware.

__Note: Your current IP adress should be 192.168.8.* Vacuum is on 192.168.8.1 . If your IP address is not as mentioned, you may need to renew DHCP address (https://www.computerhope.com/issues/ch001078.htm)__

**IMPORTANT:** The flasher scripts will create a web server where the robot will download the firmware image. This means it needs to be able to connect to port 80 on this box. Make sure that the firewall will not block the network access.

__Note: Flasher script creates web server using Python. I had troubles with this approach because it crashed several times, like this: https://github.com/dgiese/dustcloud/issues/151 I recommend installing webserver on virtual machine - like nginx (https://www.cyberciti.biz/faq/howto-install-setup-nginx-on-debian-linux-9/). After nginx is set up, copy the firmware package there. Copy it to following path (you may need to create missing subfolders): /var/www/html/image/output/. You will also need to change flasher.py (see Note3)__

__Note2: You may encounter Python error on line 52 in flasher.py. This is due to incorrect encoding settings of python. You can fix it or simply comment line 52 in flasher.py. It is only print statement of current status.__

------

__Note3: If you choose to use nginx as webserver, you need to changed flasher.py as following. First make a copy of flasher.py like flasher-nginx.py. Then comment following lines: 168,169,170,172,173,174,175,200__
Example of lines 166-203 in flasher-nginx.py:
```
#    request_handler = http.server.SimpleHTTPRequestHandler
#    httpd = ThreadedHTTPServer(('', 0), request_handler)
#    http_port = httpd.server_address[1]
    http_port = 80
#    print('Starting local http server...')
#    thread = threading.Thread(target=httpd.handle_request, daemon=True)
#    thread.start()
#    print('Serving http server at {}:{}'.format(local_ip, http_port))

    ota_params = {
        'mode': 'normal',
        'install': '1',
        'app_url': 'http://{ip}:{port}/{fw}'.format(ip=local_ip, port=http_port, fw=firmware),
        'file_md5': md5(firmware),
        'proc': 'dnld install'
    }
    print('Sending ota command with parameters:', json.dumps(ota_params))
    r = vacuum.send('miIO.ota', ota_params)
    if r[0] == 'ok':
        print('Ota started!')
    else:
        print('Got error response for ota command:', r)
        exit()

    ota_progress = 0
    while ota_progress < 100:
        ota_state = vacuum.send('miIO.get_ota_state')[0]
        ota_progress = vacuum.send('miIO.get_ota_progress')[0]
        printProgressBar(ota_progress, 100, prefix = 'Progress:', length = 50)
        sleep(2)
    print('Firmware downloaded successfully.')

#    httpd.server_close()

    print('Exiting.')
    exit()
```
------

Flash the firmware using:

```
python3 dustcloud/devices/xiaomi.vacuum/firmwarebuilder/flasher.py -f image/output/v11_001712.pkg
```

__Note: Transfer may take several minutes. After executing the previous command, vacuum will start blinking. Wait until vacuum says: "Updating firmware...."__

__Note2: If you interrupt the connection or the connection drops, nothing really happens. You can reinit the transfer by executing the previous command. Just make sure, that you do not power off the vaccum after vacuum says: "Updating firmware..."__

__Note3: There might be a problem with UTF encoding. See following page for more informaton how to fix it: https://webkul.com/blog/setup-locale-python3/__

After the successful transfer of the image to the robot, the robot will start flashing the image. This will take about 5~10 minutes. After the process is done, the robot will state that the update was successful.

Eventually, we can log in to our robot via ssh:

```
ssh root@192.168.8.1
```

If everything works out, you can now log in to your robot - enjoy.

If you face any trouble, check: "Flashing doesn't work"

## Check if dummycloud is correctly set up
Assuming you are logged into your robot, we can check if the dummycloud is correctly set up.
First, we need to examine if the dummycloud executable is running via
```
pgrep dummycloud
```

The result should be a process id number, e.g. `397`.
Next, we have to check the firewall rules, the port `8053` needs to be redirected to`127.0.0.1`, so that the robot is communicating with the dummycloud.
Execute
```
iptables -t nat -L
```
The result should contain the following:
```
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DNAT       udp  --  anywhere             ot.io.mi.com         udp dpt:8053 to:127.0.0.1:8053
```
Furthermore, one can verify if the dummycloud is working directly on the robot. If the WiFi LED is constantly on (~30 s after startup), everything is working as expected.

## Appendix

### Flashing doesn't work

Make sure your firewall is turned off that the robot can connect to your machine for downloading the firmware.

If you know your firewall can't be the reason for it, you may just try it again in the first place.

### Get robot token
Execute the following on your host PC (**not** on the robot) to obtain the robot's token:
```
cd rockrobo
source flasher/venv/bin/activate
mirobo --debug discover --handshake true
```

The token will be sth. like `456b567549786e6e73754374323*****`. 

### Connect robot to your WiFi
__Make sure you have active DHCP on your router!__

There are two ways. If you have installed Valetudo on the vacuum, you can use the UI to setup the WiFi. Simply navigate to Settings -> WiFi and change Wifi Settings and click on Save new Wifi configuration. When robot is connected to the Wifi, use your router DHCP to obtain new IP address. 

Second way is to use mirobo command:
To integrate the unprovisionised robot to your local WiFi, you need to execute the following on your host PC (**not** on the robot) to obtain the robot's token:
```
cd rockrobo
source flasher/venv/bin/activate
mirobo --debug discover --handshake true
```

The token will be sth. like `456b567549786e6e73754374323*****`. 
With this token we can set up the WiFi to which the robot will connect.
```
mirobo --ip=192.168.8.1 --token=456b567549786e6e73754374323***** configure-wifi YOUR_SSID YOUR_WIFI_PASSWORD     
```
Now the robot will be in your local WiFi and you can give it for example a static ip via your router.

### Blocking traffic to the outside of your WiFi

To make sure that no traffic from the robot can escape your local WiFi, you can add the following rules to the firewall of your robot (to make it permanent put in in /etc/rc.local before `exit0`):

```
iptables -A OUTPUT -d 192.168.0.0/16 -j ACCEPT
iptables -A OUTPUT -d 127.0.0.0/8 -j ACCEPT
iptables -A OUTPUT -j DROP
```

### Backup files ###
_Based on: https://github.com/dgiese/dustcloud/wiki/Files-to-be-backed-up_

These information should be backed up for later use and in case something goes wrong:
```
/mnt/default/device.conf
/mnt/default/vinda
/mnt/default/rockrobo.conf (if existing)
/run/shm/sn
```

──────────────────────────────────────────────────────────────

## Just for fun stuff

### Editor

If you're familiar with bash and want to play a bit more around, I can give you some advices.

Let's start getting an editor you can at least work with:

```
apt update && apt install vim
```

### Then let's fix the apt sources:

```
echo "deb http://ports.ubuntu.com/ubuntu-ports/ trusty main universe restricted multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ trusty-security main universe restricted multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ trusty-updates main universe restricted multiverse
deb http://ppa.launchpad.net/ubuntu-toolchain-r/test/ubuntu trusty main" > /etc/apt/sources.list
```

Let's then clean the apt lists and update again:

```
rm -rf /var/lib/apt/lists && apt update
```

Now looking at "df -h" will most likely release to you that there is almost no storage.
So making some storage "the dirty way": (in this case I decided to free some space to install docker and try my setup from my dedicated host)

```
mkdir -p /mnt/data/relocated/var/cache && mkdir /mnt/data/relocated/var/lib

mv /var/cache/apt /mnt/data/relocated/var/cache && ln -s /mnt/data/relocated/var/cache/apt /var/cache/apt

mv /var/lib/apt /mnt/data/relocated/var/lib && ln -s /mnt/data/relocated/var/lib/apt /var/lib/apt

mv /usr/local/bin/valetudo /mnt/data/valetudo/ && ln -s /mnt/data/valetudo/valetudo /usr/local/bin/valetudo

mv /usr /mnt/data/relocated && ln -s /mnt/data/relocated/usr /usr
```

### Upgrading

You may want to do an upgrade and have some tools which could be useful, at least for the update process of valetudo itself which I just did yesterday:

```
apt upgrade; apt install locate wget; updatedb
```

### Changing port of valetudo:

```
echo "env VAC_WEBPORT=8080" >> /etc/init/valetudo.conf && service valetudo restart
```

Note that your web panel is accessible via "http://roborock:8080".

# How-To: Rooted Xiaomi Vacuum robot Gen 1 Home Assistant connection #

First you need to have Home Assistant up and running. If you do not have Home Assistant, follow this guide (https://www.home-assistant.io/getting-started/) or if you wish to use Hassbian follow this guide (https://www.home-assistant.io/docs/installation/hassbian/installation/)

I have installed Hassbian because I wanted to have generic Raspberry Rassbian with Home Assistant installed. 
For Hassbian you may need this list of commen tasks (https://www.home-assistant.io/docs/installation/hassbian/common-tasks/)

## Setting up the connetion to vacuum ##

To set up the connection, you need to add the vacuum to the Home Assistant configuration.yaml file (on Hassbian it can be found here: /home/homeassistant/.homeassistant/configuration.yaml). There you need to add following component: _xiaomi_miio_

Xiaomi miio documentation: https://www.home-assistant.io/components/vacuum.xiaomi_miio/ 

Simply open configuration.yaml file in text editor like vim and add following lines: 

```
vacuum:
  - platform: xiaomi_miio
    host: VACUUM_IP_ADDRESS
    token: YOUR_TOKEN
```

* VACUUM_IP_ADDRESS is IP address of your vacuum in your network (it needs to be connected in your network - see Appendix -> Connect robot to your WiFi for more information)
* YOUR_TOKEN can be obtained several ways:
    * If you have Valetudo on your vacuum, you can access it using the web browser. Then navigate to Settings -> Token
    * If you do **not** have Valetudo on your vacuum follow instructions in Appendix -> Get robot token
    * Use MIIO command line tool. If you have NPM installed, you can simply install miio package and run miio discover command (https://www.home-assistant.io/components/vacuum.xiaomi_miio/#retrieving-the-access-token)

Restart Home Assistant

```
$ sudo systemctl restart home-assistant@homeassistant.service
```
Login to the Home Assistant Lovelace UI
Now you should see Xiaomi vacuum on your home screen (only if you have active automatic UI - this is active out-of-the-box)
