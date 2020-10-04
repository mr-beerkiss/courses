# KALI Notes

Running KALI on parallels so needed to do a bit of work to get everything working

## WIFI Adapteer

The ALFA AWUS036AC doesn't work out of the box despite being detected by `dmesg`. The drivers can 
be [manually installed](https://github.com/aircrack-ng/rtl8812au). The `DKMS` option worked well.

### `iw`/`ip` over `iwconfig`/`ipconfig`

It seems like iw/ip are meant to replace their `*config` variants. Here are some useful commands for the
newer commands.

```sh
# List WIFI devices
sudo iw dev

# Take interface down
sudo ip link set wlan0 down

# Rename the device (optional)
sudo ip link wlan0 set name mon0

# Set monitor mode
sudo iw dev mon0 set type monitor

# Bring the interface back up
sudo ip link set mon0 up
```

To kill problematic interference in monitor mode run `airmon-ng check kill`. Note that this will sometimes
kill the network manager. To bring it back, run `sudo systemcl start NetworkManager`.

## Packet Sniffing

To run basic packet sniffing 

```
sudo airodump-ng wlan0
```

By default, airodump only sniffs 2.4Ghz networks. To sniff on 5Ghz networks, use the `--band` argument

```
sudo airodump-ng --band a wlan0
```

To sniff both, use `--band abg`

Once you have identified a target network, you can use airodump-ng to sniff a particular network for clients

```
# bssid is the network to sniff
# channel should be specified othere airodump will scan all channels
# write will write results to a file
airodump-ng --bssid 24:C9:A1:01:EF:98 --channel 6 --write test wlan0
```

## Deauthentication Attack

You can disconnect any client from any network.

```sh
# DeauthPackets -> Number of deauth packets to send
aireplay-ng --deauth [#DeauthPackets] -a [NetworkMAC] -c [TargetMAC] [Interface]
```

## Craking WEP

WEP using RC4 encryption with includes a key and a 24-bit initialisation vector (IV). The IV is too small
and given a large enough number of collected IVs, you can crack the key.

Use `airodump-ng` with the `--write` option to get a `.cap` file which is needed by `aircrack-ng`. Given 
about 30k+ (depends on WEP key length) data packets on `airodump-ng` you can kill it and pass the `.cap` file to `aircrack-ng`. This
should give you the WEP key. Note that the default format is colons separated bytes. Remove the colons 
to get the key and try connect to the AP.

### Cracking WEP on quiet networks

If the network is quite, you can use a `--fakeauth` attack with `aireplay-ng`. This will allow you to 
associate with the AP without actually connecting it to the network. Think of it as that initially
connection that requires you to enter a password to continue.

Examples
```sh
# Fakeauth attack (associates to network)
aireplay --fakeauth 0 -a <bssid> -h <iface_mac> mon0

# ARP Replay attack (generates IVs which helps with key cracking
aireplay --arpreplay -b <bssid> -h <iface_mac> mon0
```

Once you have connected with `--fakeauth`, you can use and `--arpreplay` attack with `aireplay-ng` to
force the AP to generate a sufficient number of IVs that will allow you to crack the key with 
`aircrack-ng`.

> *NOTE* You don't actually need to kill `aireplay-ng` or `airodump-ng` before running `aircrack-ng`. In 
fact it is probably better to leave them running in case you need to generate more data.

> *NOTE* It seems that the fakeauth association and the aireplay are not enough on their own to force
ARP packets to be generated in sufficient quantities. I worked around this by connecting a device to the AP.
Not sure if there are workarounds


## WPA/WPA2 Cracking

### WPS Shortcut

Lol exploit WPS. Only works if the router is configured _not_ use PBC (Push Button Authentication).

WPS pin is only 8 digits.

Use `wash --interface <device>` to scan for APs that offer WPS. Note the `Lck` field which indicates 
whether or not the AP will lock you out after a certain number of failed attempts.

Once you have identified a target network, you can use `reaver` to try brute-force the WPS pin. Apparently
the latest version of reaver is a bit buggy. There is a download link in the course but honestly the 
download site looks very suss. 

With reaver, it's association funcionality can be a bit buggy. Best to pass the `--no-associate` flag and
use a `--fakeauth` attack with `aireplay-ng` instead.

### Capturing the Handshake

Use `airodump-ng` to capture packets from an AP. You'll need to wait for a client to connect to the 
network... or, you could use a deauthentication attack to force a client off the network and then stop
the attack which will allow them to reconnect. Just don't hammer the network with deauth packets!

Once the client reconnects you `airodump-ng` should capture the handshake (make sure you are using 
`--write`). The handshake contains data that allows you validate the key, allowing you to perform
a dictionary attack against the handshake packets to the discover the network key!

Use `aircrack-ng` to crack the MIC inside the handshake packet. Definitely worth some 
[further reading](https://www.wifi-professionals.com/2019/01/4-way-handshake)

#### Creating Word Lists with `crunch`

Create your own word lists with crunch. Check `man crunch` for examples. The `-t` option is useful if you
saw some typing a password and you heard six keypresses and the first char was `a` and the last was `s`
you can pass `-t a@@@@s`. When using `-t` the `min` and `max` args must be the same.



