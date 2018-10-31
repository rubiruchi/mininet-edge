# Mininet test for Edge network

Follow the [instructions](http://mininet.org/download/) for installing Mininet.
A minimal environment comprises:
    
    $ sudo apt-get install mininet 

Remember to use Python v2 instead of v3, since Mininet API are provided for v2 only.

Run this program with:
    
    $ sudo python topology.py
       
In order to simulate client's traffic run the `client-traffic.py` too.

Note, if during the execution submit.py script crashes for some reason or you terminate it using CTRL+C, 
make sure to clean mininet environment using:

    $ sudo mn -c
    
    
## Random topology

Before executing the Mininet script, create a python file capable of creating a 
random topology with `N` switches, `E` links, `H` hosts and `MAX_BW` in the links, 
attached to the controller at IP `Controller_IP`
run the following commands:

    $ gcc -o random-topology random-topology.c
    $ ./random-topology N E H MAX_BW Controller_IP
    
For instance, a possible configuration could be:

    $ ./random-topology 10 10 10 100 127.0.0.1
    
Before executing the program. remember to start the SDN controller(e.g., Ryu).
Now deploy the topology on Mininet:

    $ sudo python test_topology.py
    
After running the script, you can test the behaviour, here some tips:

    mininet> h1 ping h2
    mininet> nodes
    mininet> net
    mininet> dump
    
    mininet> h1 python -m SimpleHTTPServer 80 &
    mininet> h2 wget -O - h1
    ...
    mininet> h1 kill %python
    
    mininet> exit
    
## Testbed

`edge_topology_test.py` is an example of a file generated by random generator, 
and improved to allow links redundancy in some switches. 
It was used in some experiments as testbed.

## Make Mininet network communicates with Internet

If you want to allow your netwrok to communicate with the external world, 2 solutions are showed:

1. Configure NAT

You can add NAT to the network via code:
    
    nat0 = net.addNAT('nat0', ip=natIP, localIntf='eth0', inNamespace=False)
    net.addLink(nat0 , s1)
    
Then the hosts supposed to send packets over Internet need to configure a DNS server.
It can be done adding the DNS IP, e.g., 8.8.8.8, in the `/etc/resolv.conf` file. For instance:

    h1 echo "nameserver 8.8.8.8" > /etc/resolv.conf
    
2. Attach Mininet to loacl host:

Make sure that your guest OS, i.e., mininet OS, is connected to the internet.

Run the Mininet program, either via `sudo mn ...`  or `sudo python ...`.

Check the openvswitch configuration using the command:

    ovs-vsctl show
    
Now, run the following command to connect a new interface (eth1) to one of the switches in your network (s1): 

    ovs-vsctl add-port s1 eth1

Check the configuration again using `ovs-vsctl show`, you should note the new interface.

Then run dhclient on the hosts supposed to communicate over Internet. For instance for h1:

    h1 ifconfig h1-eth0 0
    h1 dhclient h1-eth0

The first command removes the ip from h1-eth0,the second command gets the ip address for h1-eth0 from dhcp server. 

Now check the internet connectivity using ping.

    h1 ping www.google.com
    
Please be careful as in this solution the IP address of the hosts is assigned by DHCP, thus this can break the controller behavior.

## Performance Evaluation:

The idea is to simulate real traffic in the edge in addition to the data related to the 
`Livemicro` application.

The real traffic is emulated using real traces in `unina.pcap`. 
You can run the following command for sending packets based on this trace.

    h7 tcpreplay --intf1=h7-eth0 unina.pcap
