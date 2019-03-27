### Initial System Setup

1: These steps use fresh install of Raspbian Stretch Lite
2: Update and upgrade
    sudo apt-get update
    sudo apt-get dist-upgrade
3. Try the upgrade a second time

If you need to build 8812au drivers first, see [Building realtek 8812 drivers](https://github.com/joseananio/linux-hacks/tree/master/raspberrypi/building-realtek-8812audrivers.md).

### Setting Up Bridge-like Link Between eth0 and wlan0

The aim is to setup a bridge between the our wireless adapter and the ethernet port. The wifi adapter connects to a wifi. We want to share that connection with the ethernet port. We can then setup another network on the ethernet port.

It is assumed that you have wlan0 and eth0 as your interfaces. If not replace them in the commands.

We will setup a dhcp server on the eth0 interface to allow connections from whatever device we plug in. 

Taken from: https://pimylifeup.com/raspberry-pi-wifi-bridge/

1. install dnsmasq.
    ```
    sudo apt-get install dnsmasq
    sudo systemctl stop dnsmasq
    ```

2. setup the `wlan0` connection
    ```sh
    sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
    ```
    add to end of the file:
    ```sh
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=DE #replace with country code
    network={
            ssid="yourwifinetworkname"
            psk="yourwifinetworkpassword",
            priority=100
    }
    ```

3. Set up `wlan0` for the wifi network
    ``` 
    sudo vim /etc/network/interfaces
    ```
    add this to end of the file
    ```sh
    auto wlan0
    allow-hotplug wlan0
    iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    # you can set upo static ip
    ```

4. set up `eth0` interface; for dhcp server
    ```
    sudo vim /etc/network/interfaces
    ```
     add to end of the file
    ```sh
    auto eth0
    iface eth0 inet static
    address 192.168.220.1   # eth0 serves as the gateway for the dnsserver
    netmask 255.255.255.0
    ```


5. Disable dhcpcd: Strech dhcpcd does not work well with */etc/network/interfaces*
    ```
    sudo systemctl disable dhcpcd
    ```

6. Setup `dnsmasq`: the dns server on `eth0`
    ```
    sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig  #backup original
    sudo vim /etc/dnsmasq.conf  #open new for editing
    ```
    add this to the end of the file.
    ```sh
    interface=eth0       # Use interface eth0  
    listen-address=192.168.220.1   # Specify the address to listen on (eth0) 
    bind-interfaces      # Bind to the interface
    server=8.8.8.8       # Use Google DNS  or any
    domain-needed        # Don't forward short names  
    bogus-priv           # Drop the non-routed address spaces.  
    dhcp-range=192.168.220.50,192.168.220.150,12h # IP range and lease time  
    ```

7. Enable ip forwarding
    ```sh
    sudo vim /etc/sysctl.conf
    ```
    find and uncomment *#net.ipv4.ip_forward=1* like this.
    ```sh
    net.ipv4.ip_forward=1 
    ```
    enable the change immediately without reboot
    ```sh
    sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"  
    ```

8. configure the Raspberry Pi’s firewall to forward traffic from eth0 wlan0.
    ```
    sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE  
    sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
    sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT  
    ```
    **NOTE: reboot if you encounter errors and try again**

    Save the iptable rules 
    ```sh
    sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
    ```
    Enable loading the rules on every reboot
    ```
    sudo vim /etc/rc.local
    ```
    add this above the `exit 0` line
    ```sh
    iptables-restore < /etc/iptables.ipv4.nat
    ```

11. Start the dnsmasq service
    ```
    sudo service dnsmasq start
    ```

12. Reboot the system
    ```
    sudo reboot
    ```
#### Tips
If networking.service complains of `/var/run/wpa_supplicant/wlan0`
remove /var/run/wpa_supplicant/wlan0 

#### Fix dnsmasq restart failures on boot
dnsmasq may fail to start. This means the local network connection will not work. here is a quick fix
Set up autologin and execute a command in `~/.bashrc`
You can use other autostart methods that execute after dnsmasq has failed.

Taken from : https://stackoverflow.com/questions/47594337/how-can-i-auto-login-with-the-latest-raspbian-stretch-lite

1. Create service configuration file:
    ```
    sudo vim /etc/systemd/system/getty@tty1.service.d/autologin.conf
    ```
    enter:
    ```
    [Service]
    ExecStart=
    ExecStart=-/sbin/agetty --autologin <user> --noclear %I 38400 linux
    ```
    Note: *replace `<user>` with your pi username*

2. Reboot
    ```
    sudo systemctl enable getty@tty1.service
    ```