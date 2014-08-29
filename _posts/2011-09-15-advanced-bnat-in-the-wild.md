---
title: "Advanced BNAT in the Wild"
layout: post
---

Just this week, we were asked to help out with some “TCP weirdness” that was identified out on a customer site during a penetration test.   A port was identified as open, but when attempting to connect to the port, the connection was immediately RST by the client (not the server).

Let’s take a look below at an example packet capture of what he was seeing when trying to connect to a file share…

    **SYN**
    ab:cd:ef:01:01:01 > ab:cd:ef:02:02:02, ethertype IPv4 (0x0800), length 54: 10.31.1.182.29387 > 10.176.232.91.135: S 1527382016:1527382016(0) win 3276811:55:35.421521

    **SYN/ACK**
    ab:cd:ef:03:03:03 > ab:cd:ef:01:01:01, ethertype IPv4 (0x0800), length 60: 10.176.232.91.135 > 10.31.1.182.29387: S 2422132266:2422132266(0) ack 1887842761

    **RST**
    ab:cd:ef:01:01:01 > ab:cd:ef:02:02:02, ethertype IPv4 (0x0800), length 54: 10.31.1.182.29387 > 10.176.232.91.135: R 1887842761:1887842761(0) win 0

There are two issues here that could be the reason why the TCP communication channel is bombing out for the user…

1. The acknowledgement number (1887842761) in the SYN/ACK packet is not the expected 1527382017 (ISN + 1) that we should have gotten when we sent an ISN of 1527382016 in the SYN.
2. The MAC address that we are sending the SYN to (ab:cd:ef:02:02:02) and the MAC address that we are getting the SYN/ACK from (ab:cd:ef:03:03:03) are different.

Why does this happen? Well, what happened in this case was that in this network not only is there asymmetric routing (noted by the presence of two routers in this communication), but in addition we are seeing a variance in SEQ numbers as the packet was mangled as it traversed it’s routing loop through the network to the server and back to the initiating client.

How could we fix this issue from an attackers perspective? Oddly enough, the answer happens to dovetail quite well into a talk I gave at [DC Skytalks](http://skytalks.info/) this year while at DEFCON 19 on “BNAT Hijacking: Repairing Broken Communication Channels”.  BNAT, which stands for Broken Network Address Translation, is a phenomenon that occurs when packets are modified in transit to the point at which the communication channel becomes unusable by the client and/or server.  As a result, modern port and vulnerability scanners do not detect these services as reachable leaving a perfectly good service hidden from view.

In most BNAT cases (such as the case above) the communication channel is actually terminated as a result of your client not successfully understanding the response you are getting from the server resulting in a RST.  In response to this and to enable security professionals to overcome obstacles such these, I created a suite of tools in Ruby that can allow you to redesign an in connection and successfully interact with the service called “BNAT-Suite”.

What was really interesting about the above mentioned case was that it is not a very common method of BNAT.  In my research and discussed in my recent talk, the more common form of BNAT is actually an IP mismatch, rather than a sequence number mismatch.  In this case, although the MAC addresses are mismatched (point #2 above), this actually does not affect TCP handshake process at all.  This means that the only thing that stops this communication from successfully handshaking is the mangled sequence number and everything else is perfectly valid.

In my quest to prove that most BNAT’d services can be successfully reconstructed, I mocked up an example of how we can successfully complete the TCP 3-way handshake with a new tool I've added to the project called bnat-handshake.  This tool is aimed at helping security professionals better understand the mechanics of BNAT at the most basic level while also proving that advanced scenarios can be successfully redesigned.

The way bnat-handshake works is that we start out by building a raw SYN packet in Ruby using Packetfu (a gem created by [@todb](https://twitter.com/#!/todb) that is used heavily in the Metasploit project)…

    $config = PacketFu::Utils.whoami?(:iface=>"#{$int}")
    synpkt = PacketFu::TCPPacket.new(:config=>$config, :timeout=> 0.1, :flavor=>"Windows")
    synpkt.ip_saddr=$config[:ip_saddr]
    synpkt.ip_daddr="#{$target}"
    synpkt.tcp_sport=rand(64511)+1024
    synpkt.tcp_dport=$port.to_i
    synpkt.tcp_win=14600
    synpkt.tcp_options="MSS:1460,SACKOK,TS:3853;0,NOP,WS:5"
    synpkt.eth_saddr=$config[:eth_saddr]
    synpkt.eth_daddr=$gatewaymac
    synpkt.tcp_flags.syn=1
    synpkt.recalc

We then start a packet capture so that we can successfully catch any mangled SYN/ACK responses…

    cap = PacketFu::Capture.new(
            :iface => $config[:iface],
            :start => true,
            :filter => "tcp and dst #{$config[:ip_saddr]} and tcp[13] == 18"
          )

We then place the SYN packet on the wire…

    synpkt.to_w

And then we process each SYN/ACK packet that comes back and matches the filter we defined in our capture…

    listen = Thread.new do
      loop {
        cap.stream.each {|pkt| packet = PacketFu::Packet.parse(pkt)
          puts "got the syn/ack"
          ackpkt = TCPPacket.new(
                    :config=>$config,
                    :timeout=> 0.1,
                    :flavor=>"Windows"
                   )
          ackpkt.ip_saddr=packet.ip_daddr
          ackpkt.ip_daddr=packet.ip_saddr
          ackpkt.eth_saddr=packet.eth_daddr
          ackpkt.eth_daddr=packet.eth_saddr
          ackpkt.tcp_sport=packet.tcp_dport
          ackpkt.tcp_dport=packet.tcp_sport
          ackpkt.tcp_flags.syn=0
          ackpkt.tcp_flags.ack=1
          ackpkt.tcp_ack=packet.tcp_seq+1
          ackpkt.tcp_seq=packet.tcp_ack
          ackpkt.tcp_win=183
          ackpkt.recalc
          ackpkt.to_w
          puts "sent the ack"
        }
      }
    end
    listen.join

The last block of code is what you might consider the first major ingredient in the “secret sauce” of BNAT.  What we do is very simply ignore the original state of the original SYN and only focus on what the SYN/ACK is expecting of us by “reflectively ack’ing” the SYN/ACK in an exact mirror response which successfully completes the TCP 3-way handshake.

The second major ingredient in BNAT, that needs to happen before you send all the above mentioned raw packets, is suppressing the RST your client is bound to send when receiving response traffic that doesn’t perfectly match the request.  One way to suppress the RST and allow you to perform this packet “kung-fu” is using IPTables.

This command will suppress packets with the RST flag set and help you continue the session where other clients would have failed…

    iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP

So needless to say, I think this stuff is pretty cool.  I’ve enumerated a short list of possible BNAT scenarios below that I believe you can successfully complete the 3-way handshake and prove that it’s possible to redesign the TCP session and make it viable by simply tweaking how the client responds…

- IP Address only
- Port Number only
- Sequence Number only
- IP Address + Port Number
- IP Address + Sequence Number
- Port Number + Sequence Number
- IP Address + Port Number + Sequence Number

In addition to simply completing the handshake for all of these with bnat-handshake, the end goal is handling all these scenarios supported with the bnat-router (a tool to make these scenarios transparent to the end user like a pentester/vulnerability scanner to interact with a BNAT’d service).  Currently, the bnat-router has full session support for IP only BNAT, but I suspect you’ll see more support over time and I flesh out these use cases and improve the code.

The video below is a video I created to show the potential impact of BNAT services and mentioned in a recent post on the Metasploit community blog ([link](https://community.rapid7.com/community/metasploit/blog/2011/08/26/a-tale-from-defcon-and-the-fun-of-bnat)) a couple weeks back.   The video shows how the BNAT auxiliary modules combined with an exploit module could help you exploit a BNAT’d (aka: “hidden”) service.

<iframe width="560" height="315" src="//www.youtube.com/embed/FS_cg1PVhkI" frameborder="0" allowfullscreen></iframe>

If you are interested in learning more about the BNAT project, you can find the BNAT code up on GitHub ([https://github.com/claudijd/bnat](https://github.com/claudijd/bnat)) or use the ip-based modules I co-wrote with [@msfbannedit](https://twitter.com/#!/msfbannedit) that are now included in the Metasploit framework ([http://metasploit.com/](http://metasploit.com/)) under the auxiliary modules section.

This was also posted on the SpiderLabs blog [here](http://blog.spiderlabs.com/2011/09/advanced-bnat-broken-network-address-translation-in-the-wild.html).