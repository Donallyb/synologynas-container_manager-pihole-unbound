# Running Pi-Hole + Unbound on a Synology NAS using Container Manager (Docker)


## Description:

This project deploys Pi-Hole and Unbound DNS resolver using docker-compose on a Synology NAS. The configurations create both a MacVLAN and a custom bridge network for the containers, offering flexibility in managing network interactions. By employing this setup, we aim to alleviate the challenges in ensuring that Pi-Hole and Unbound start in the proper sequence, which can affect MacVLAN IP assignments.

## Why use Pi-Hole + Unbound on a Synology NAS?

1. ### Network-Level Ad Blocking with Pi-hole:

* Blocks ads on all network-connected devices.
* Boosts page loading speeds and reduces data usage.
* Evades many "ad-blocker detected" prompts on websites.

2. ### Privacy and Control:

* Pi-hole halts various trackers, safeguarding user privacy.
* Its admin interface offers insights into network queries, allowing domain whitelisting or blacklisting.

3. ### Enhanced DNS with Unbound:

* Provides faster local DNS resolution without relying on third-party servers.
* Enhances privacy by keeping DNS queries in-house.
* Supports DNSSEC for verified and secure DNS responses.

4. ### Optimized Synology NAS Usage:

* The NAS's always-on nature ensures continuous ad-blocking and DNS services.
* Synology's Docker support simplifies deployment and management of Pi-hole and Unbound.

5. ### Cost Efficiency:

* Uses the Synology NAS to avoid additional hardware costs.
* Centralizes management, simplifying network architecture.

## Prerequisites:

* Ensure you have installed the Container Manager package (formerly known as Docker) on your Synology NAS.
* Install the Git Server package on your NAS.
* Activate SSH on your Synology NAS.

## Setup Steps:

1. SSH into your Synology NAS using:
```bash
ssh [admin account]@[NAS IP Address]
```
2. Navigate to the /volume1/docker directory:
```bash
cd /volume1/docker
```
3. Clone this repository:
```bash
sudo git clone https://github.com/Donallyb/synologynas-container_manager-pihole-unbound.git
```
4. Navigate into the cloned repository directory:
```bash
cd /synologynas-container_manager-pihole-unbound
```
5. Set up host volume mount points for Pi-Hole:
```bash
mkdir -p pihole/pihole; mkdir -p pihole/dnsmasq.d
```
6. Edit the .env file to match your specific environment settings. The values in the .env file will be utilized to auto-populate the docker-compose.yaml file.
```bash
# Pi-hole specific configurations
HOSTNAME=<Enter the hostname you would like to assign to the Pi-hole container>
PIHOLE_MACVLAN_IP=<Enter the first assignable IP address from the /30 MacVLAN subnet you created>
PIHOLE_BRIDGE_IP=<Enter the one assignable IP address from the /32 custom bridge subnet you created>
WEBPASSWORD='<Enter the Pi-Hole GUI password you would like to use (keep the single quotes)>'

# Unbound DNS specific configurations
UNBOUND_MACVLAN_IP=<Enter the second assignable IP address from the /30 MacVLAN subnet you created>

# Network configurations
NETWORK_INTERFACE=<Enter the network interface your Synology NAS uses to access your LAN>

# Volume Mapping Configurations
PIHOLE_CONFIG_PATH=/volume1/docker/synologynas-container_manager-pihole-unbound/pihole/pihole
DNSMASQ_CONFIG_PATH=/volume1/docker/synologynas-container_manager-pihole-unbound/pihole/dnsmasq.d

# Macvlan network settings
MACVLAN_SUBNET=<Enter the LAN network>/24
MACVLAN_GATEWAY=<Enter the LAN Gateway>
MACVLAN_IP_RANGE=<Enter the starting IP address of the LAN subnet that the MacVLAN network will use>/30

# Bridge network settings
BRIDGE_SUBNET=<Enter the custom bridge network>/24
BRIDGE_GATEWAY=<Enter the custom bridge network gateway>
BRIDGE_IP_RANGE=<Enter the IP address of the custom bridge network>/32

# Container Timezone
TIMEZONE=<Enter the timezone you are in>
```
7. Deploy the Pi-Hole and Unbound containers and set up the MacVLAN and custom bridge networks:
```bash
sudo docker-compose up -d
```

## Usage:
You can access the Pi-Hole Web GUI using the MacVLAN IP assigned to it.
The Pi-Hole admin interface can be accessed using the password specified in the .env file.
Use the Pi-Hole MacVLAN IP as the DNS server for your client devices or DHCP server.
You can also designate the Pi-Hole custom bridge network IP on your Synology NAS.

## References:

Inspired by the setup from [digtalaloha](https://github.com/digtalaloha/synology-docker-pihole-unbound).
### Further reading:
[Pi-Hole with Unbound on Docker](https://github.com/chriscrowe/docker-pihole-unbound/)
[Pi-Hole Docker Official] (https://github.com/pi-hole/docker-pi-hole#quick-start)
