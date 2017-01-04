# bramble
This repository will outline the steps I've used to build a Raspberry
Pi cluster. The process will be as automated and as documented as possible.

## Naming
* `pi-plate` (master/host)
* `pi-slice-01` (client/slave)
* `pi-slice-02` (client/slave)
* `pi-slice-03` (client/slave)
* `pi-slice-04` (client/slave)
* `pi-slice-05` (client/slave)

## Materials
Below is the list materials I've used. Many variants are possible

* 6 [Raspberry Pi 1 Model B boards](https://www.raspberrypi.org/products/model-b/)
* 6 SanDisk 8 GB SD cards
* USB 2.0 Ethernet adapter (optional)
* 8-port TP Link Ethernet switch
* Powered USB strip
* 6 6-in. Ethernet cables
* 6 1-ft. micro-USB cables

## Preparing the images
Each card is running the [Raspbian Jessie Lite OS](https://www.raspberrypi.org/downloads/raspbian/). It is convenient to configure one base image for all of the cards and just flash each accordingly. Below is a checklist for configuring this base image:

* Enable SSH
* Set timezone appropriately
* Set keyboard to English (US)
* Set the hostname appropriately

## DHCP Host/Client Configuration
We need to configure one Pi as a DHCP host (`pi-plate`). I've used the instructions found [here](https://www.raspberrypi.org/learning/networking-lessons/lesson-3/plan/).

### Host
On `pi-plate`, install `dnsmasq`,
```shell
$ sudo apt-get update
$ sudo apt-get install dnsmasq
```
Then edit the `/etc/network/interfaces` file to include the lines,
```
auto eth0
iface eth0 inet static
address 192.168.0.1
netmask 255.255.255.0
```
where `eth0` is the default ethernet port. If you're using the port exposed by our USB ethernet adapter, this is probably `eth1`. Then restart the networking service,
```shell
$ sudo service networking restart
```
Finally, change the dnsmasq config file, `/etc/dnsmasq.conf`, to,
```
interface=eth0
dhcp-range=192.168.0.2,192.168.0.254,255.255.255.0,12h
```
and restart the dnsmasq service,
```shell
$ sudo service dnsmasq restart
```

### Client
On the client Pi's, we just need to alter the `/etc/network/interfaces` file to look to the DHCP server running on the host,
```
iface eth0 inet dhcp
```
and then restart the networking service again.

At this point, the client Pi's are allocated an IP address by the host Pi, but only the host is connected to the outside world. To forward the connection from the host to the clients, on the host, first enable IPv4 forwarding,
```
$ sudo sysctl -w net.ipv4.ip_forward=1
```
and enable NAT,
```
$ sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

## Ansible
[Ansible](http://docs.ansible.com/ansible/) is a tool for managing many servers easily. We only need to install Ansible on the host,

```shell
$ sudo apt-get install ansible
```

## Shared Filesystem

## TORQUE/PBS
* On the head node
  * Download torque (from Adaptive computing)
  * Need libssl-dev and libxml2-dev to configure
  * Unzip and run ./configure, make, make install
  * Create the server pbs_server -t create
  * Set all of the necessary settings using qmgr
  * Set master and slaves to all have static IP addresses
  * Configure hostname and node names in /etc/hosts file (can’t use IP addresses)
  * make sure that they all have the same /etc/hosts  
  * Configure keyless ssh between master and all slaves
  * Run the make packages in the unzipped torque folder
  * copy the relevant packages to the slaves
  * install the slave packages and start the MOM
  * after making sure the hostnames match, add the node on the master
  * restart the server; repeat for all nodes
  * configure NFS by creating a shared drive on the master node and mounting it on each of the slaves, making sure that the paths on both are equivalent

## Additional Software
Software to be installed on each node of the Raspberry bramble:

* gcc
* gfortran
* lapack
* octave
* python
  * ipython
  * matplotlib
  * numpy
  * scipy
* torque
* root
* git
* nfs
