# Networking Basics and Wireshark

Presentation: [Networking Basics and Wireshark](networking-basics-and-wireshark.pdf)

Activity: Wireshark Challenge

## Challenge Walkthrough

1. Packet Count
	10
2. First Request
	93.184.216.34
3. Server Banner
	nginx/1.18.0 (Ubuntu)
4. Full URI
	http://example.com/index.html
5. Protocol Breakdown
	8:2 TCP:HTTP
6. Largest Packet
	243
7. Query Count
	18
8. Record Type
	Packet 3, Type A
9. Busiest Host
	Either, it's a conversation
10. Response Time
	512ms
11. Query Pattern
	Repeated requests to data.exfil-test.net with garbage requests suggests DNS exfil of data
12. Beacon Domain
	cdn-update-service.net, 14  times
13. Dropped File
	svchost_update.exe, application/octet-stream
14. Something Extra
	cookie src={3-byte data}
	combine into a single string in packet order
	decode from base64
	flag{c00k1e_dr1p_3xf1l}
15. Incident Summary
	10.0.0.23 got infected with a C2 beacon, which exfiltrated the flag data via http cookies to 198.51.100.77 while beaconing.
	At time 41.8269, infected host pulls a malicous executible "svchost_update.exe" from 198.51.100.77.