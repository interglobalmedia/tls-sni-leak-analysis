# Use

## Generating traffic to capture packets

The `sudo tcpdump -i eth0 -s 65535 -w capture.pcap` command runs silently in the background, so the next thing for me to do was to generate traffic. I visited my Personal Blog (`mariadcampbell.com`) and my project hosted on GitHub Pages called Guess the Keys.

## Stopping the tcpdump packet capture command with Ctrl + C

When I stopped the `sudo tcpdump -i eth0 -s 65535 -w capture.pcap` command, it returned:

```shell
^C
47008 packets captured
47008 packets received by filter
0 packets dropped by kernel
```

I captured 47008 packets in a matter of a few minutes, and 47008 packets were received by filter. And 0 packets were dropped by the Kali Linux kernel. This was a very good thing!

## Opening capture.pcap in Wireshark

Next, I opened the `capture.pcap` file in Wireshark by running:

```shell
wireshark capture.pcap
```

This led me to:

![Screenshot of the wireshark capture.pcap command: it opened the capture.pcap file in the Wireshark GUI](../images/Screenshot-2026-08-14-at-9.25.47-AM.jpg)

_The wireshark capture.pcap command: it opened the capture.pcap file in the Wireshark GUI_

## Going to Statistics → Protocol Hierarchy

The first thing I wanted to do was go to Statistics → Protocol Hierarchy in Wireshark. Why?

Protocol hierarchy shows me a breakdown of all protocols in a capture, listing each protocol's packet count and byte percentage. This helps quickly identify what types of traffic dominate my network.

### Looking for specific protocols such as HTTP and DNS

![Screenshot of Protocol Hierarchy in Wireshark](../images/Screenshot-2026-08-15-at-6.21.34-AM.jpg)

_The Protocol Hierarchy in Wireshark_

![Screenshot of the rest of Protocol Hierarchy in Wireshark (overlaps a bit)](../images/Screenshot-2026-08-15-at-6.40.24-AM.jpg)

_The rest of Protocol Hierarchy in Wireshark (overlaps a bit)_

### Finding resolved addresses in capture.pcap

To find resolved addresses in my `capture.pcap` file in Wireshark, first I go into `Edit → Preferences → Name Resolution` and then check "Use captured DNS packet data for name resolution".

![Screenshot of Wireshark Preferences with "Use captured DNS packet data for name resolution" checked](../images/Screenshot-2026-08-15-at-10.56.48-AM.jpg)

_Wireshark Preferences with "Use captured DNS packet data for name resolution" checked_

That's the specific option that tells Wireshark to scan the DNS response packets already in my `capture.pcap` and use them to populate the resolved-address table.

### Going to Statistics -> Resolved Addresses

After enabling "Use captured DNS packet data for name resolution" in Wireshark preferences, I had to reload the selected capture. This meant going to `View → Reload Ctrl key + R key`. I had to reload the selected capture before the DNS-sourced hostnames actually appeared. Checking the box alone didn't retroactively apply to packets already dissected in the open `capture.pcap` file. 

Now when I went into `Statistics → Resolved Addresses`, I saw those hostnames show up in the Hosts tab alongside the MAC/multicast entries:

![Screenshot of IP Addresses and their hostnames in Wireshark](../images/Screenshot-2026-08-15-at-11.44.43-AM.jpg)

_IP Addresses[^3] and their hostnames in Wireshark_

Why did I have to select Wireshark Preferences with "Use captured DNS packet data for name resolution" in `Preferences → Name Resolution`? If I didn't turn on the "Use captured DNS packet data for name resolution", Wireshark would not utilize any DNS responses that were captured in the file. This means that any hostname resolution that could have been derived from these responses would be unavailable.

Selecting "Use captured DNS packet data for name resolution" option persists across sessions, which means that any capture I started from then on would already have "Use captured DNS packet data for name resolution" from the very first dissection pass. Packet dissection in Wireshark is crucial because it allows the software to decode and analyze the details of network packets, enabling me to understand the data being transmitted. 

### Searching for HTTP protocols in Wireshark

To find only HTTP protocols in my capture.pcap file in Wireshark, I typed http in the Filter Search bar:

![Screenshot of filtering for HTTP protocols in the capture.pcap file in Wireshark](../images/Screenshot-2026-08-15-at-12.12.27-PM.jpg)

_Filtering for HTTP protocols in the capture.pcap file in Wireshark_

## Differences between HTTP and DNS protocols in Wireshark

| HTTP Protocol | DNS Protocol |
| --- | --- |
| Deals with the transmission of web documents between a client and a server, using methods like GET and POST | Resolves domain names to IP addresses |
| HTTP packets show request and response details | DNS packets focus on name resolution queries and responses. |

---

[^3]: An IP address is a numerical address, such as 192.168.1.1, assigned to a device on an IP network, so other devices can identify it. It has two main functions: identifying a device's network interface and helping route traffic to its location on the network.

---

[← Back to index](README.md)
