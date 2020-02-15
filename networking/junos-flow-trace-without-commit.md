# Junos flow trace without commit

On SRX, there is now a handy feature introduced in 12.1X46-D10. You can enable flow trace without going into configuration on the operational mode. I believe this will make troubleshooting easier as it saves time if you need to try different flow filters. Here is how you can enable a sample ICMP flow trace for a specific IP address e.g 192.168.1.10 Create your filters named incoming-filter,outgoing-filter to catch the traffic

> monitor security flow filter incoming-filter protocol icmp destination-prefix 192.168.1.10 monitor security flow filter outgoing-filter protocol icmp source-prefix 192.168.1.10 monitor security flow filter incoming-filter protocol icmp destination-prefix 192.168.1.10 monitor security flow filter outgoing-filter protocol icmp source-prefix 192.168.1.10 Give a file name to save the flow trace monitor security flow file flow-trace.log monitor security flow file flow-trace.log File will be saved under /var/log folder, you can also set size option if you like Check the filters show monitor security flow Monitor security flow session status: Inactive \`Monitor security flow trace file: /var/log/flow-trace.log Monitor security flow filters: 2 Name: incoming-filter Status: Inactive Source: 0.0.0.0/0 \(port 0~65535\) Destination: 192.168.1.10/32 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None Name: outgoing-filter Status: Inactive Source: 192.168.1.10/32 \(port 0~65535\) Destination: 0.0.0.0/0 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None show monitor security flow  
> Monitor security flow session status: Inactive Monitor security flow trace file: /var/log/flow-trace.log Monitor security flow filters: 2 Name: incoming-filter Status: Inactive Source: 0.0.0.0/0 \(port 0~65535\) Destination: 192.168.1.10/32 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None Name: outgoing-filter Status: Inactive Source: 192.168.1.10/32 \(port 0~65535\) Destination: 0.0.0.0/0 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None

Yes we have created the filters but they are not active as you can see on the Status field. Until you start monitoring nothing is being traced. Start the trace:

> monitor security flow start monitor security flow start

We can see that now filters are active

> show monitor security flow Monitor security flow session status: Active Monitor security flow trace file: /var/log/flow-trace.log Monitor security flow filters: 2 Name: incoming-filter Status: Active Source: 0.0.0.0/0 \(port 065535\) Protocol: icmp Logical system: root-logical-system Interface: None Name: outgoing-filter Status: Active Source: 192.168.1.10/32 \(port 065535\) Protocol: icmp Logical system: root-logical-system Interface: None show monitor security flow Monitor security flow session status: Active Monitor security flow trace file: /var/log/flow-trace.log Monitor security flow filters: 2 Name: incoming-filter Status: Active Source: 0.0.0.0/0 \(port 0~65535\) Destination: 192.168.1.10/32 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None Name: outgoing-filter Status: Active Source: 192.168.1.10/32 \(port 0~65535\) Destination: 0.0.0.0/0 \(port 0~65535\) Protocol: icmp Logical system: root-logical-system Interface: None

Generate the traffic and check the log file

> 'show log flow-trace.log Jul 10 20:48:10 20:48:09.965641:CID-0:RT:&lt;192.168.1.1/1-&gt;192.168.1.10/29284;1&gt; matched filter incoming-filter: Jul 10 20:48:10 20:48:09.965941:CID-0:RT:packet \[84\] ipid = 38310, @0x4bd2c4d2 Jul 10 20:48:10 20:48:09.965964:CID-0:RT:—— flow\_process\_pkt: \(thd 1\): flow\_ctxt type 15, common flag 0x0, mbuf 0x4bd2c280, rtbl\_idx = 0 Jul 10 20:48:10 20:48:09.970789:CID-0:RT: flow process pak fast ifl 72 in\_ifp ge-0/0/1.0 Jul 10 20:48:10 20:48:09.970793:CID-0:RT: ge-0/0/1.0:192.168.1.1-&gt;192.168.1.10, icmp, \(8/0\) Jul 10 20:48:10 20:48:09.970801:CID-0:RT: find flow: table 0x58735f30, hash 9701\(0xffff\), sa 192.168.1.1, da 192.168.1.10, sp 1, dp 29284, proto 1, tok 6 Jul 10 20:48:10 20:48:09.971211:CID-0:RT: no session found, start first path. in\_tunnel - 0x0, from\_cp\_flag
>
> show log flow-trace.log  
> Jul 10 20:48:10 20:48:09.965641:CID-0:RT:&lt;192.168.1.1/1-&gt;192.168.1.10/29284;1&gt; matched filter incoming-filter:

Jul 10 20:48:10 20:48:09.965941:CID-0:RT:packet \[84\] ipid = 38310, @0x4bd2c4d2

Jul 10 20:48:10 20:48:09.965964:CID-0:RT:—— flow\_process\_pkt: \(thd 1\): flow\_ctxt type 15, common flag 0x0, mbuf 0x4bd2c280, rtbl\_idx = 0

Jul 10 20:48:10 20:48:09.970789:CID-0:RT: flow process pak fast ifl 72 in\_ifp ge-0/0/1.0

Jul 10 20:48:10 20:48:09.970793:CID-0:RT: ge-0/0/1.0:192.168.1.1-&gt;192.168.1.10, icmp, \(8/0\)

Jul 10 20:48:10 20:48:09.970801:CID-0:RT: find flow: table 0x58735f30, hash 9701\(0xffff\), sa 192.168.1.1, da 192.168.1.10, sp 1, dp 29284, proto 1, tok 6

Jul 10 20:48:10 20:48:09.971211:CID-0:RT: no session found, start first path. in\_tunnel - 0x0, from\_cp\_flag - 0

Yes we have caught the traffic Now it is time to stop the monitoring and clearing the filters

> monitor security flow stop &gt;clear monitor security flow filter incoming-filter &gt;clear monitor security flow filter outgoing-filter monitor security flow stop clear monitor security flow filter incoming-filter clear monitor security flow filter outgoing-filter

All done!

