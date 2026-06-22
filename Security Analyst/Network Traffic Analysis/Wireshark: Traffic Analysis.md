### Nmap Scans
Q1: What is the total number of the "TCP Connect" scans?
1. Use the given "~/Desktop/exercise-pcaps/nmap/Exercise.pcapng"
2. Use the query: *tcp.flags.syn==1 and tcp.flags.ack ==0 and tcp.window_size > 1024*\

Q2: Which scan type is used to scan the TCP port 80?
1. Relies on the three-way handshake (needs to finish the handshake process).
2. Usually conducted with nmap -sT command.
3. Used by non-privileged users (only option for a non-root user).
4. Usually has a windows size larger than 1024 bytes as the request expects some data due to the nature of the protocol.\

Q3: How many "UDP close port" messages are there?
1. Use filter: *icmp.type==3 and icmp.code==3* \
   
Q4: Which UDP port in the 55-70 port range is open?
1. Filter: *udp.dstport in {55..70}*
2. Analyze the wireshark notice that only one port that are not currently in use.


### ARP Poisoning and MiTM
What is the number of ARP requests crafted by the attacker?
What is the number of HTTP packets received by the attacker?
What is the number of sniffed username&password entries?
What is the password of the "Client986"?
What is the comment provided by the "Client354"?






