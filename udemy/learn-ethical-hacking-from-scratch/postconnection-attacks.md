# Post Connection Attacks

## Information Gathering

Once you have gained access to a network, the next step is to perform information gathering to find out
what resources are on the network you have just gained access to.

## `netdiscover`

`netdiscover` is a very easy to use tool for scanning networks you may have gained access to.

How, easy? This easy

```sh
netdiscover -r <cidr range>
```

## `nmap`

- [`nmap` book]() recommended

### `zenmap`

`zenmap` is the offical GUI for `nmap` but from what I can tell it is no longer has active maintainers
and has been removed rom the kali apt repository. The `nmap` website offers a `rpm` package binary
which you can convert to `.deb` package using a tool calleed `alien`

`zenmap` is good for learning sicne it offers a whole bunch of built in profiles and it will show you
the nmap command it will run so you can copy & paste it to a terminal window. The output log also has some
nice coloureed output making it a bit easy to ready.

`Ping scan` is a good tool to quickly find hosts, but a lot of hosts might block ICMP packets so it isn't
very reliable. `Quick scan` will go a step further and show you open ports on all the hosts `nmap` 
discovers. Finally, `Quick scan plus` will go one step even further and try to work out the names and 
versions response for the open ports on that machine.

## Man in the middle (MITM) attacks

### ARP Spoofing

- ARP -> Address Resolution Protocol
  - maps IP address of machine to its MAC address
  - ARP is a broadcast message used to find the owner of an IP address and respond with its MAC Address
  - hosts usually store an ARP table on their machine for a local mapping of IP and MAC addresses
    - use `arp -a` to list the contents of the table
- An ARP spoofing attack is one where the attack exploits the ARP protocol by
  - deceiving the AP into believing the attack machine is the target machine
  - deceiving the target into believing the attacker machine is the AP
- Why do ARP Spoofing attacks work?
  - clients accept responses even if they didn't request them
  - clients trust a response with no verification step
- How to run
  - `arpspoof` - very simple tool
  - part of the `dsniff` package (not installed in 2020.3 by default)

#### Example

```sh
# Tell the target machine you are the router
arpspoof -i <iface> -t <target_ip> <spoof_address>

# Tell the router you are the target
arpspoof -i <iface> -t <router_ip> <host_address>

# The above command will cause traffic to flow from the target to the host. IP forwarding 
# is disabled by default. It can easily be enabled by running the following
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

> *NOTE* The course is running as route and uses basic redirection. Since modern Kali doesn't use root user
> you need to run [`tee` instead](https://askubuntu.com/questions/783017/bash-proc-sys-net-ipv4-ip-forward-permission-denied)


