## Converting Oracle SBC (formerly AcmePacket SBC) log files to Wireshark PCAP files
[![Build Status](https://travis-ci.org/fran-ovia/apktlog2pcap.svg?branch=master)](https://travis-ci.org/fran-ovia/apktlog2pcap)

1. It is a simple application to convert SBC log files to Wireshark PCAP files.
2. No special requirement! Just Java Runtime Environment installed in your PC.
3. This apktlog2pcap tool is intended to work as a replacement for the old log2cap tool (formerly provided by AcmePacket and now discontinued). It also adds some new features:
   * The tool does not assume all SIP messages are SIP over UDP, but analyses Via header to infer and use the actual tranport protocol (UDP, TCP or SCTP)
   * Conversion to PCAP file is also available for non-network events (any log line in log.sipd, log.mbcd, log.algd). Here the text content is encapsulated in a Syslog message

## How to use apktlog2pcap?

First o all, got to this repository's releases section (https://github.com/fran-ovia/apktlog2pcap/releases) and download apktlog2pcap.jar and (optionally) apktlog2pcap.bat too.

There are two ways of using apktlog2pcap:

**A. Just double-clicking on apktlog2pcap.jar file:**

The app will then look for SBC log files (sipmsg.log*, log.sipd*, log.mbcd*, log.algd*) in the same directory and convert them to PCAP files (sipmsg.log.pcap, log.sipd.pcap, log.mbcd.pcap, log.algd.pcap).

A command-line-like window will be opened to summarize the results of the process.

**B. Invoking apktlog2pcap.bat script from command line:**

Options are explained in the help:

```
#> apktlog2pcap.bat -h
apktlog2pcap.v0.9.0.build20170426:

Usage 1 (converts the input log file into the output PCAP file):

    apktlog2pcap -f <input_file> <output_file>

Usage 2 (converts the log files from the input directory into PCAP files in t
he output directory):

    apktlog2pcap -d <input_directory> <output_directory>
```

Note that you might need to edit apktlog2pcap.bat script to customize the locations of your java.exe executable (if it's not already included in your PATH variable) and apktlog2pcap.jar file (if you don't want to store it in the same directory as the apktlog2pcap.bat script).

Note that I'm not including an .sh script equivalent to the .bat script, since implementing it is so straightforward and would probably need customization anyway.

## Why use log files to generate PCAP files instead of simply capturing the network traffic (as it can be done with packet-trace command)?

1. Because of SIP over TLS: if we just capture the network traffic, in order to analyze the SIP messages we need to decrypt the TLS traffic, which is cumbersome when not impossible:
   * Not always we have the SBC's certificate's secret key
   * Sometimes the trace starts after the TLS handshake has been completed...
   * Even if the previous points were not a problem for us, we might eventually need to pass the trace to other person (such as the one providing support to a CPE sending faulty SIP traffic over TLS), but then we will not want to share our SBC's certificate keys necessary to decrypt the TLS traffic...
However, since all SIP messages (even those sent over TLS) are present in sipmsg.log, we can just get sipmsg.log and use apktlog2pcap to convert it into a PCAP file (in such case the SIP messages will be included in TCP frames)
2. So we can have non-network log events (such as logs at log.sipd) in a PCAP file, then merge them with other PCAP file containing network traffic, and then use the merged PCAP file to easily analyse the SBC system events in the context of the network traffic
3. Because it is a valid option for people without admin rights (not necessary to FTP download SBC logs), whereas executing packet-trace does require admin rights 

## Some explanations on the SBC log files and apktlog2pcap application

1. First of all, note that I have found no specification of the Oracle SBC log file format. Since it is a textual format, it has been quite easy to implement a parser for the log files I've seen so far, but It might happen that log files generated by other SBC models / firmware versions are not parsed correctly by apktlog2pcap. I will try to keep apktlo2pcap updated to support all of them though.
2. The SIP network messages stored in the SBC log do not contain the whole network packet, but just the SIP, IPs, ports and VLAN ID. Thus, when generating the PCAP file apktlog2pcap assigns default values for the rest of the network fields not provided in the log file (such as Ethernet MAC addresses and flags from link, network and trasnsport layers).
