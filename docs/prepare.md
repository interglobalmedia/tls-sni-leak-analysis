# Prepare

## Set Up My Environment

1. Install `tcpdump` and `Wireshark` on my system.
1. Ensure I have the necessary permissions to capture network traffic.

## Capture Network Traffic with tcpdump

- Use `tcpdump` to capture packets from a specific network interface[^1].

### Example command

```shell
sudo tcpdump -i <interface> -s 65535 -w capture.pcap
```

This command captures all packets and saves them to a file named `capture.pcap`.

## Analyze Captured Traffic with Wireshark

- Open the `capture.pcap` file in Wireshark.
- Use Wireshark's GUI to filter and analyze the packets.
- Look for specific protocols, such as HTTP or DNS, to understand traffic behavior.
- Decrypt Encrypted Traffic (Optional)
    - If capturing HTTPS traffic, set the SSLKEYLOGFILE[^2] environment variable before launching the browser, so it logs TLS session keys as it negotiates each connection.
    - Point Wireshark at that key log file (Preferences → Protocols → TLS → (Pre)-Master-Secret log filename) to decrypt the matching capture.

## Project Goals

- Understand how different network protocols operate.
- Diagnose connectivity issues by analyzing packet flow.
- Monitor traffic behavior to identify potential security threats.
- This project will provide practical experience with both tools, enhancing my skills in network analysis and cybersecurity.

## Checking for available interfaces

To check for interfaces available to me, I ran:

```shell
sudo tcpdump -D
```

The `-D` flag, short for  `--list-interfaces`, lists the network interfaces available on my system and where `tcpdump` can capture packets.

The command yielded:

```shell
sudo tcpdump -D  

[sudo] password for maria: 

1.eth0 [Up, Running, Connected]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
5.nflog (Linux netfilter log (NFLOG) interface) [none]
6.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
7.dbus-system (D-Bus system bus) [none]
8.dbus-session (D-Bus session bus) [none]
```

The logical network interface to select was `eth0`. It is my real interface and the one to capture on.

## Starting the capture

To start a capture, I ran:

```shell
sudo tcpdump -i eth0 -s 65535 -w capture.pcap
```

The command provided me with the following information:

```shell
sudo tcpdump -i eth0 -s 65535 -w capture.pcap

tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 65535 bytes
```

it told me that I was listening on the eth0 network inferface as I had requested.

The `-i` flag, short for `--interface`, was followed by the name of the network interface I wanted to target (`eth0`).

The `-s` flag, is short for `--snapshot-length`. According to `man tcpdump`, 

> Snarf snaplen are bytes of data from each packet rather than the default of 262144 bytes. Packets truncated because of a limited snapshot are indicated in the output with ``[|proto]'', where proto is the name of the protocol level at which the truncation has occurred.

The number `65535` represents the number of bytes of data returned from the `-s` flag truncation.

The `-w` flag, writes the raw packets to a file instead of parsing and printing them out. They, however, can later be printed with the `-r` option.

`capture.pcap` is the file `tcpdump` writes the raw packets to.

---

[^1]: A network interface is where a computer and network connect on Linux.

[^2]: `SSLKEYLOGFILE` is used to specify a file where TLS secrets are logged, allowing tools like Wireshark to decrypt HTTPS traffic. It must be set before starting applications like browsers or curl to capture the necessary key log data.

---

[← Back to index](README.md)
