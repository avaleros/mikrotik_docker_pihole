# Pihole docker on Mikrotik v7.5
First of all, we need to install Docker package from https://mikrotik.com/download.

We will check if docker service is active. On Mikrotik console:

`/system/device-mode/print`


To activate it:

`/system/device-mode/update container=yes`


Let's to configure the url where our Mikrotik will download the containers:

`/container/config/set registry-url=https://registry-1.docker.io tmpdir=usb/docker/tmp`


Create a Bridge and assign an IP:

`/interface/bridge/add name=docker`

`/ip/address/add address=172.16.0.1/29 interface=docker`

Create the masquerade firewall rule

`/ip/firewall/nat/add chain=srcnat action=masquerade src-address=172.16.0.0/29`


Create the virtual interface of our docker, asign an ip and add to the bridge:

`/interface/veth/add name=veth-pihole address=172.16.0.2/29 gateway=172.16.0.1`

`/interface/bridge/port add bridge=docker interface=veth-pihole`


Define variables

`/container/envs/add name=pihole key=TZ value="Europe/Madrid"`

`/container/envs/add name=pihole key=WEBPASSWORD value=webpasswd`

`/container/envs/add name=pihole key=DNSMASQ_USER value=root`


Create mount points

`/container/mounts/add name=pihole_etc src=usb/mounts/pihole/etc dst=/etc/pihole`

`/container/mounts/add name=pihole_etc src=usb/mounts/pihole/dnsmasq.d dst=/etc/dnsmasq.d`


On Winbox create a new container and write:

Remote image: 

Interface: veth-pihole

Hostname: pihole

Root Dir: usb/docker/containers/pihole

Mounts: pihole_etc
        pihole_dnsmasq.d


Now, start the container and modify your DHCP server. Set the ip of veth-iphole the new DNS on network.

## Schedule a startup

Find the name of your docker:

`/container/print`

Open a new scheduler and write:

`:delay 5;`

`container/start [find where name="NAME OF YOUR DOCKER"]`

## More
https://hub.docker.com/r/pihole/pihole/

https://github.com/pi-hole/docker-pi-hole#readme
