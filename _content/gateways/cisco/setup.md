---
title: Setup
zindex: 800
---

# Setup

Before starting the gateway, you'll need to set it up. If you've never used Cisco equipment before, we're providing you with basic instructions to set up the gateway, and basic system and network configuration.

## Gateway setup

* Attach the **antennas** that you have for the gateway. If you have 2 antennas, plug in the both of them.

![](antenna.jpg)

* Plug in the **GPS antenna** for the gateway.

![](gps.jpg)

* Plug in the **Ethernet cable** of the gateway to your own switch or point of entry.

![](ethernet.jpg)

* Plug in the **power cable** for the gateway. The Cisco LoRaWAN gateway supports two power methods:

    * Using a 48 VDC adapter, to plug in the *Power* port ;

    * Using Power over Ethernet if you have a PoE adapter. In that case, use the adapter on the Ethernet port of the gateway.

* Finally, to access the Cisco console, you'll need a console cable from USB to RJ45. Plug the RJ45 end in the *Console* port of the gateway, and the USB port to your computer.

Your gateway is now up and running, and connected to your computer!

## Configuration

### Connect to the Cisco gateway console

You now have a serial connection to the Cisco gateway's console.

To open up a terminal, you have several approaches available:

+ Either use specialized serial connection software, such as [Serial](https://www.decisivetactics.com/products/serial/) for macOS.

+ Either use your operating system's native serial connection software. For example, on macOS and Linux, open a Terminal, and type the following commands:

    ```bash
    ls /dev/tty.usb*
    ```


    You'll see the list of available USB serial devices. Once you've found the once matching the Cisco console, connect using the following command:

    ```bash
    screen <device> 115200 # e.g. screen /dev/tty.usbserial-AO001X6M 115200
    ```
    +For Windows, you can download Putty or any serial software, Select Serial COM based on your Console-USB, and make sure to use 115200 speed



### System setup

This part covers **setting up basic system and network parameters**: updating it, connecting it to a local network, setting up `ntp` and the GPS. If you have specific network constraints, check with your system or network administrator how to set up the system for your network.

You'll notice that the user shell is quite different from Unix-based shells. This shell is called **standalone mode**, and is meant for a basic usage of the gateway. In many cases, the standalone mode shell won't let you input unexpected characters and entries. In any shell, you can use the `?` command to show the list of available commands.

First, type `root` to login as the privileged user, and enter `enable` to turn on privileged commands. To verify that privileged commands are enabled, check that the prompt is `Gateway#` and not `Gateway>`.

Note: When purchasing a new IXM gateway, make sure to order it as a standalone unit, otherwise you will have to purchase an IR809 router to convert the IXM to standalone, there is no other way to do it. to convert the IXM to a standalone using the IR809 router you will have to connect the IXM as a vitrual interface "VLPWA" on the router and then access it to conver it. please refer to the following configuration guide on how to access the IXM as a virtual interface https://www.cisco.com/c/en/us/td/docs/routers/access/800/829/software/configuration/guide/b_IR800config/b_vlpwa.html

#### Update

To update your Cisco gateway, you can download the latest [firmware update here](https://software.cisco.com/download/home/286311296/type/286311234/release/2.0.30?imageguid=A74F81D27FD3C1DF311D59849D054C10468D338E) - at least version 2.0.30. You will need a Cisco account with the right permissions to download the update file. If your account does not have those permissions, get in touch with your Cisco representative or with Cisco support.

Once you've retrieved the update file, you can update your gateway by several ways: USB, FTP or TFTP. We'll cover in this guide how to update over USB. You will need for this a USB stick formatted in FAT. The USB port of the Cisco IXM being tiny, you'll want to use the tiniest USB stick that you own.

1. Copy the update `*.tar.gz` file as `cisco_update.tar.gz` file at the root of the USB stick.

2. Plug the USB stick in the Cisco gateway.

3. Execute the following commands:
    
    ```bash
    usb enable
    archive download-sw firmware /normal /save-reload usb:cisco_update.tar.gz
    ```

This will update your Cisco firmware. After the operation, you can verify the installed version with `show version`.

#### Network setup

To configure your Cisco Gateway to your network, type the following commands:

```bash
configure terminal # Enter global configuration mode
interface FastEthernet 0/1 # Enter Ethernet configuration
```

If your local network has a **DHCP server** attributing IPs:

```bash
ip address dhcp
```

Otherwise, if you know the **static IP address** of your gateway:

```bash
ip address <ip-address> <subnet-mask>
```

Next, type the following to save the network configuration of your gateway:

```bash
description Ethernet # Replace "Ethernet" by any description you want
exit # Exit Ethernet configuration
exit # Exit global configuration mode
copy running-config startup-config # Save the configuration
```

You can test your Internet configuration with the `ping` command, for example:

```bash
ping ip 8.8.8.8 # Ping Google's DNS server
```

To see more information about the gateway's IP and the network, you can use `show interfaces FastEthernet 0/1`, `show ip interfaces FastEthernet 0/1` or `show ip route`.

#### Date and time

To configure your system's date and time, you can use `ntp`:

```bash
configure terminal # Enter global configuration mode
ntp server address <NTP server address> # Configure ntp on an ntp address
# OR
ntp server ip <NTP server IP> # Configure ntp on an IP

exit # Exit global configuration mode
```

If you don't have production-grade `ntp` servers available, you can use [`pool.ntp.org`](http://www.pool.ntp.org/en/use.html)'s servers.

#### FPGA

If you needed to update your gateway firmware previously, your FPGA will need ~20 minutes to update once the new firmware is installed. The packet forwarder will not work until then, so we recommend at this point waiting until the FPGA is upgraded.

To show the status of the FPGA, you can use the following command:

```bash
show inventory
```

When the `FPGAStatus` line indicates `Ready`, this means you can go forward with this guide.

#### GPS

If you have a GPS connected to your Cisco gateway, enable it with the following commands:

```bash
configure terminal # Enter global configuration mode
gps ubx enable # Enable GPS
exit # Exit global configuration mode
```

#### Enable radio

As a final step before setting up the packet forwarder software, we're going to **enable the radio**. You can see radio information with the `show radio` command:

```
Gateway#show radio 
LORA_SN: FOC21028R8S
LORA_PN: 95.1602T01
LORA_SKU: 915
LORA_CALC: <NA,NA,NA,50,31,106,97,88,80,71,63,53,44,34,25,16-NA,NA,NA,54,36,109,100,91,83,74,66,57,48,39,30,21>
CAL_TEMP_CELSIUS: 31
CAL_TEMP_CODE_AD9361: 87
RSSI_OFFSET: -204.00,-204.40
LORA_REVISION_NUM: C0
RSSI_OFFSET_AUS: -203.00,-204.00

radio status: 
on
```

If the radio is off, enable with it `no radio off`.

> 📜 The `show radio` command also shows you more information about the LoRa concentrator powering the gateway. For example, `LORA_SKU` indicates the base frequency of the concentrator.

### Enable authentication

To prevent unauthorized access to the gateway, you'll want to set up user authentication. The Cisco gateway has a **secret** system, that requires users to enter a secret to access privileged commands.

To enable this secret system, you can use the following commands:

+ `configure terminal` to enter global configuration mode.
+ To set the secret, you can use different commands:
	+ `enable secret <secret>` to enter in plaintext the secret you wish to set, instead of `<secret>`. *Note*: Special characters cannot be used in plain secrets.
	+ `enable secret 5 <secret>` to enter the secret **md5-encrypted**, instead of `<secret>`.
	+ `enable secret 8 <secret>` to enter the secret **SHA512-encrypted**, instead of `<secret>`.
+ `exit` to exit global configuration mode.
+ `copy running-config startup-config` to save the configuration.

### Verifications

Before we install the packet forwarder, let's run verifications to ensure that the gateway is ready.

+ Type `show radio` to verify that the **radio is enabled**. The result should indicate `radio status: on`.
+ Type `show inventory` to verify that the **`FPGAStatus` is `Ready`**.
+ Type `show gps status` to verify that the **GPS is correctly connected**. You can get additional GPS metadata by typing `show gps info`.
+ Verify that the **network connection is working**. You can test this by pinging common ping servers with `ping ip <IP>`, if your local network does not block ping commands. For example, you can ping Google's servers with `ping ip 8.8.8.8`.

If some of those checks fail, go back to the appropriate section earlier in order to fix it.

## Installing the packet forwarder

> ⚠️ Keep in mind that the pre-installed packet forwarder is not supported by Cisco for production purposes.

To run the packet forwarder, we'll make use of the **container** that is running on the gateway at all times.

### Packet forwarder configuration

When in standalone mode, enter the container:

```bash
request shell container-console
```

Move to the directory where the packet forwarder configuration is located, and show the list of available frequency plans:

```bash
cd /etc/pktfwd
ls -la
```

Depending on your frequency plan, or on your specific setup, you'll want to use the most appropriate frequency plan. Once you've decided on a frequency plan, set it as your current configuration:

```bash
mv config.json config_old.json
mv <your config> config.json
```

We're now going to modify the configuration to point the packet forwarder to The Things Network. Select the [router address](../packet-forwarder/semtech-udp.md#router-addresses) the most appropriate for your location.

Edit the configuration using a text editor, such as `vi`:

```bash
vi config.json
```

You'll need to change the following settings:

* `server_address` to **the router address** (such as `router.eu.thethings.network`)

* `serv_port_up` and `serv_port_down` to `1700`

Write down the value written for `gateway_ID`. Save the file, and exit your text editor.

### Gateway registration

Don't close the shell yet, but open a Web browser and head to your the Things Network console. Click on **Gateways** > **Register gateway**. Check the *I'm using legacy packet forwarder* box, and enter the gateway EUI that was indicated as `gateway_ID` in the configuration. Fill in the rest of the form, and click on **Register gateway**.

> ⚠️ If the gateway EUI is already marked as used, you'll need to change the `gateway_ID` value in the configuration of the gateway. The value must be an 8 bytes hexadecimal value.

### Testing the packet forwarder

You can now test the packet forwarder by executing:

```bash
pktfwd -c config.json -g/dev/ttyS1
```

You should now see packets flowing in the logs of the packet forwarder - and see gateway activity on the gateway's package on the console!

> Having issues making the packet forwarder run? Head over to our [packet forwarder troubleshooting section](../troubleshooting/semtech-udp.md).

### Installing the packet forwarder

Now that we know the packet forwarder is running, let's make it run permanently.

Save the following script as `/etc/init.d/S90_pkt_forwarder.sh`:

```bash
#!/bin/sh

SCRIPT_DIR=/etc/pktfwd
SCRIPT=pkt_forwarder
RUNAS=root

PIDFILE=$SCRIPT_DIR/pkt_forwarder.pid

start() {
  if [ -f /var/run/$PIDNAME ] && kill -0 $(cat /var/run/$PIDNAME); then
    echo 'Service already running' >&2
    return 1
  fi
  echo 'Starting service…' >&2
  cd $SCRIPT_DIR
  start-stop-daemon -S -q -b -p "$PIDFILE" --exec "$SCRIPT" -- -c config.json -g/dev/ttyS1
  echo 'Service started' >&2
}

stop() {
  if [ ! -f "$PIDFILE" ] || ! kill -0 $(cat "$PIDFILE"); then
    echo 'Service not running' >&2
    return 1
  fi
  echo 'Stopping service…' >&2
  kill -15 $(cat "$PIDFILE") && rm -f "$PIDFILE"
  echo 'Service stopped' >&2
}

uninstall() {
  echo -n "Are you really sure you want to uninstall this service? That cannot be undone. [yes|No] "
  local SURE
  read SURE
  if [ "$SURE" = "yes" ]; then
    stop
    rm -f "$PIDFILE"
    echo "Notice: log file is not be removed: '$LOGFILE'" >&2
    update-rc.d -f <NAME> remove
    rm -fv "$0"
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  uninstall)
    uninstall
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|uninstall}"
esac
```

This script will be called at every startup of the container. To enable it immediately, execute `./S90_pkt_forwarder.sh start`.

### Exit container shell

If you are done with configuration, you can now safely remove the serial cable between your computer.


### Reset the IXM to default:
next to the console port the reset button, keep pressing it for more than 5 seconds to reset the device to factory defaults. 
