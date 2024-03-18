# *__Network Sniffing__*

On the data-link layer lies the distinction between _switched_ and _unswitched_ networks. On an unswitched network, Ethernet packets pass through every device on the network, expecting each system device to only look at the packets sent to its destination address. However, it’s fairly trivial to set a device to _promiscuous mode_, which causes it to look at all packets, regardless of the destination address. Most packet-capturing programs, such as `tcpdump`, drop the device they are listening to into promiscuous mode by default. Promiscuous mode can be set using `ifconfig`, as seen in the following output.

<pre style="color: white;">
reader@hacking:~/booksrc $ ifconfig eth0
eth0    Link encap:Ethernet HWaddr 00:0C:29:34:61:65
        UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
        RX packets:17115 errors:0 dropped:0 overruns:0 frame:0
        TX packets:1927 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:4602913 (4.3 MiB) TX bytes:434449 (424.2 KiB)
        Interrupt:16 Base address:0x2024

reader@hacking:~/booksrc $ sudo ifconfig eth0 promisc
reader@hacking:~/booksrc $ ifconfig eth0
eth0    Link encap:Ethernet HWaddr 00:0C:29:34:61:65
        UP BROADCAST RUNNING PROMISC MULTICAST MTU:1500 Metric:1
        RX packets:17181 errors:0 dropped:0 overruns:0 frame:0
        TX packets:1927 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:4668475 (4.4 MiB) TX bytes:434449 (424.2 KiB)
        Interrupt:16 Base address:0x2024

reader@hacking:~/booksrc $
</pre>

The act of capturing packets that aren’t necessarily meant for public viewing is called _sniffing_. Sniffing packets in promiscuous mode on an unswitched network can turn up all sorts of useful information, as the following output shows.

<pre style="color: white;">
reader@hacking:~/booksrc $ sudo tcpdump -l -X 'ip host 192.168.0.118'
tcpdump: listening on eth0
21:27:44.684964 192.168.0.118.ftp > 192.168.0.193.32778: P 1:42(41) ack 1 win
17316 &lt;nop,nop,timestamp 466808 920202&gt; (DF)
0x0000 4500 005d e065 4000 8006 97ad c0a8 0076          E..].e@........v
0x0010 c0a8 00c1 0015 800a 292e 8a73 5ed4 9ce8          ........)..s^...
0x0020 8018 43a4 a12f 0000 0101 080a 0007 1f78          ..C../.........x
0x0030 000e 0a8a 3232 3020 5459 5053 6f66 7420          ....220.TYPSoft.
0x0040 4654 5020 5365 7276 6572 2030 2e39 392e          FTP.Server.0.99.
0x0050 3133 13
21:27:44.685132 192.168.0.193.32778 > 192.168.0.118.ftp: . ack 42 win 5840
&lt;nop,nop,timestamp 920662 466808&gt; (DF) [tos 0x10]
0x0000 4510 0034 966f 4000 4006 21bd c0a8 00c1          E..4.o@.@.!.....
0x0010 c0a8 0076 800a 0015 5ed4 9ce8 292e 8a9c          ...v....^...)...
0x0020 8010 16d0 81db 0000 0101 080a 000e 0c56          ...............V
0x0030 0007 1f78 ...x
21:27:52.406177 192.168.0.193.32778 > 192.168.0.118.ftp: P 1:13(12) ack 42 win
5840 &lt;nop,nop,timestamp 921434 466808&gt; (DF) [tos 0x10]
0x0000 4510 0040 9670 4000 4006 21b0 c0a8 00c1          E..@.p@.@.!.....
0x0010 c0a8 0076 800a 0015 5ed4 9ce8 292e 8a9c          ...v....^...)...
0x0020 8018 16d0 edd9 0000 0101 080a 000e 0f5a          ...............Z
0x0030 0007 1f78 5553 4552 206c 6565 6368 0d0a          ...xUSER.leech..
21:27:52.415487 192.168.0.118.ftp > 192.168.0.193.32778: P 42:76(34) ack 13
win 17304 &lt;nop,nop,timestamp 466885 921434&gt; (DF)
0x0000 4500 0056 e0ac 4000 8006 976d c0a8 0076          E..V..@....m...v
0x0010 c0a8 00c1 0015 800a 292e 8a9c 5ed4 9cf4          ........)...^...
0x0020 8018 4398 4e2c 0000 0101 080a 0007 1fc5          ..C.N,..........
0x0030 000e 0f5a 3333 3120 5061 7373 776f 7264          ...Z331.Password
0x0040 2072 6571 7569 7265 6420 666f 7220 6c65          .required.for.le
0x0050 6563 ec
21:27:52.415832 192.168.0.193.32778 > 192.168.0.118.ftp: . ack 76 win 5840
&lt;nop,nop,timestamp 921435 466885&gt; (DF) [tos 0x10]
0x0000 4510 0034 9671 4000 4006 21bb c0a8 00c1          E..4.q@.@.!.....
0x0010 c0a8 0076 800a 0015 5ed4 9cf4 292e 8abe          ...v....^...)...
0x0020 8010 16d0 7e5b 0000 0101 080a 000e 0f5b          ....~[.........[
0x0030 0007 1fc5 ....
21:27:56.155458 192.168.0.193.32778 > 192.168.0.118.ftp: P 13:27(14) ack 76
win 5840 &lt;nop,nop,timestamp 921809 466885&gt; (DF) [tos 0x10]
0x0000 4510 0042 9672 4000 4006 21ac c0a8 00c1          E..B.r@.@.!.....
0x0010 c0a8 0076 800a 0015 5ed4 9cf4 292e 8abe          ...v....^...)...
0x0020 8018 16d0 90b5 0000 0101 080a 000e 10d1          ................
0x0030 0007 1fc5 5041 5353 206c 3840 6e69 7465          ....PASS.l8@nite
0x0040 0d0a ..
21:27:56.179427 192.168.0.118.ftp > 192.168.0.193.32778: P 76:103(27) ack 27
win 17290 &lt;nop,nop,timestamp 466923 921809&gt; (DF)
0x0000 4500 004f e0cc 4000 8006 9754 c0a8 0076          E..O..@....T...v
0x0010 c0a8 00c1 0015 800a 292e 8abe 5ed4 9d02          ........)...^...
0x0020 8018 438a 4c8c 0000 0101 080a 0007 1feb          ..C.L...........
0x0030 000e 10d1 3233 3020 5573 6572 206c 6565          ....230.User.lee
0x0040 6368 206c 6f67 6765 6420 696e 2e0d 0a            ch.logged.in...
</pre>

Data transmitted over the network by services such as _telnet_, _FTP_, and _POP3_ is unencrypted. In the preceding example, the user leech is seen logging into an _FTP_ server using the password _l8@nite_. Since the authentication process during login is also unencrypted, usernames and passwords are simply contained in the data portions of the transmitted packets.

`tcpdump` is a wonderful, general-purpose packet sniffer, but there are specialized sniffing tools designed specifically to search for usernames and passwords. One notable example is Dug Song’s program, `dsniff`, which is smart enough to parse out data that looks important.

<pre style="color: white;">
reader@hacking:~/booksrc $ sudo dsniff -n
dsniff: listening on eth0
-----------------
12/10/02 21:43:21 tcp 192.168.0.193.32782 -> 192.168.0.118.21 (ftp)
USER leech
PASS l8@nite

-----------------
12/10/02 21:47:49 tcp 192.168.0.193.32785 -> 192.168.0.120.23 (telnet)
USER root
PASS 5eCr3t
</pre>