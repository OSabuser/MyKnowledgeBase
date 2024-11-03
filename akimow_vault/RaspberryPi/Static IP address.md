
# How to Assign a Static IP to a Raspberry Pi

1. **Determine your Raspberry PI's current IP v4 address** if you don't already know it. The easiest way to do this is by using the _hostname -I_ command at the command prompt. If you know its hostname, you can also ping the Pi from a different computer on the network.

```bash
hostname -I
```

![hostname -I](TuheXMwnUkyXvPxrJKLQkY-320-80.png)

2. **Get your router's IP address** if you don't already know it. The easiest way to do this is to **use the command** _**ip r**_ and take the address that appears after "default via."

```bash
ip r
```


3. **Get the IP address of your DNS** (domain name server) by enter the command below. This may or may not be the same as your router's IP. 

```bash
grep "namesever" /etc/resolv.conf
```


Now that you have the IP address your Pi is currently using, the router's IP address and the DNS IP address, you can edit the appropriate configuration file.


4. **Open /etc/dhcpcd.conf** for editing in nano.

```bash
nano /etc/dhcpcd.conf
```

5. **Add the following lines** to the bottom of the file. If such lines already exist and are not commented out, remove them.

Replace the comments in brackets in the box below with the correct information. Interface will be either _wlan0_ for Wi-Fi or _eth0_ for Ethernet.

```bash
interface [INTERFACE]
static_routers=[ROUTER IP]
static domain_name_servers=[DNS IP]
static ip_address=[STATIC IP ADDRESS YOU WANT]/24
```

In our case, it looked like this.

```bash
interface wlan0
static_routers=192.168.7.1
static domain_name_servers=192.168.1.1
static ip_address=192.168.7.121/24
```

You may wish to substitute "inform" for "static" on the last line. Using _inform_ means that the Raspberry Pi will attempt to get the IP address you requested, but if it's not available, it will choose another. If you use static, it will have no IP v4 address at all if the requested one is in use.

6. **Save the file** by hitting CTRL + X and **reboot**. 

From now on, upon each boot, the Pi will attempt to obtain the static ip address you requested.

## Using the Raspberry Pi OS Guide to Set a Static IP

If you already have all the information about your router's IP and DNS IP, you can configure the static IP address using the Network Preferences menu instead of editing the dhcpcd.conf file.

1. **Right click on the network status icon** and **select the Wireless & Wired Network Settings.**

![Select Wired & Wireless Network Settings](9ckMqBJ6oaMcs3EcRivXe6-320-80.png)


2. **Select the appropriate interface**. If you're configuring a static IP for Wi-FI, choose wlan0. For Ethernet, choose eth0.

![Select interface](YB5qWTEaSG5n7f9itqSRHB-320-80.png)


3. Enter the IP addresses into the relevant fields.  Your desired IP address will be in the IPv4 field, followed by a /24. Your router's IP and DNS server's IP will be in the fields named after them.

![Enter the appropriate IPs](uZRTVCXZb8k7wpWPM66rUL-320-80.png)


4. **Click Apply**, **close the window** and **reboot** your Pi.

Your Pi will now attempt to use your desired IP address at each boot. However, the Network Preferences menu sets this as a preference, not an absolute. So, if the IP address you asked for is not available, it will use another.


 
