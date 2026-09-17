# Networking Basics and Wireshark

Presentation: [Networking Basics and Wireshark](/presentations/Networking-Basics.pdf)

Activity: Wireshark Challenge

## Challenge Walkthrough

This walkthrough assumes you have Wireshark open with the correct file for each challenge.

1. Packet Count
Wireshark shows the total packet count in the botton right of the interface. `10`

2. First Request
Wireshark shows the destination of a packet as the fourth column by default. For the very first packet, this reads `93.184.216.34`

3. Server Banner
We see under the Protocol tab that packet 4 is HTTP. We can filter for this type by applying the display filter `http` in the bar just above the column headers.
After applying that filter, two packets are visible. The first is a GET request, and the second it's corresponding response.
Clicking on the response, the bottom half of the screen now displays the information the packet contains.
In the bottom right, Wireshark breaks down the data in a formatted dropdown menu. After expanding the `Hypertext Transfer Protocol` section, we can see: 
```
Frame 6: Packet, 243 bytes on wire (1944 bits), 243 bytes captured (1944 bits)
Ethernet II, Src: aa:bb:cc:00:00:50 (aa:bb:cc:00:00:50), Dst: aa:bb:cc:00:00:05 (aa:bb:cc:00:00:05)
Internet Protocol Version 4, Src: 93.184.216.34, Dst: 10.0.0.5
Transmission Control Protocol, Src Port: 80, Dst Port: 51000, Seq: 1, Ack: 85, Len: 189
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
    Server: nginx/1.18.0 (Ubuntu)\r\n
    Content-Type: text/html\r\n
    Content-Length: 75\r\n
    Connection: close\r\n
    \r\n
    [Request in frame: 4]
    [Time since request: 42.000000 milliseconds]
    [Request URI: /index.html]
    [Full request URI: http://example.com/index.html]
    File Data: 75 bytes
Line-based text data: text/html (1 lines)
```

Reading the Server banner, we see the answer:
`nginx/1.18.0 (Ubuntu)`

4. Full URI
This can also be seen in the above expansion.
`http://example.com/index.html`

5. Protocol Breakdown
Wireshark offers a variety of analysis tools that are very helpful when dealing with large packet captures. 
The Protocol Hierarchy view can be opened by clicking `Statistics -> Protocol Hierarchy`.

Of the 10 TCP packets in the capture, we can see that 2 of them are HTTP.

6. Largest Packet
Packet Length analysis can be viewed via `Statistics -> Packet Lengths`.
Checking the Max Val, we see `243`.

7. Query Count
Looking at the DNS queries in the capture, we can see a query number as `Standard Query: 0x10XX` in each packet, starting at 0x1001 and ending at 1012.
Since the number starts with 0x, we know that it's base16/hexadecimal. Doing some quick conversion, this means there are `18` total DNS queries, each with their own response from the DNS server. 

8. Record Type
Packet 3, Type A
Clicking on packet 3 (since we could pick any packet), we can see in the bottom left under Domain Name System:
```
Frame 3: Packet, 77 bytes on wire (616 bits), 77 bytes captured (616 bits)
Ethernet II, Src: aa:bb:cc:00:00:05 (aa:bb:cc:00:00:05), Dst: aa:bb:cc:00:00:35 (aa:bb:cc:00:00:35)
Internet Protocol Version 4, Src: 10.0.0.5, Dst: 10.0.0.1
User Datagram Protocol, Src Port: 40002, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x1002
    Flags: 0x0100 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 0
    Queries
        www.wikipedia.org: type A, class IN
            Name: www.wikipedia.org
            [Name Length: 17]
            [Label Count: 3]
            Type: A (1) (Host Address)
            Class: IN (0x0001)
    [Response In: 4]
```
Looking at `Domain Name System -> Queries -> www.wikiedia.org -> Type`, we see that the request is for a `Type A` record.


9. Busiest Host
Conversations can be viewed with `Statistics -> Conversations`. We see only one conversation between two hosts.
Since there are only two hosts, and the question asks for the most active sender OR reciever, either host would be correct. 
I'll pick busiest sender, which was `10.0.0.1`.

10. Response Time
DNS Statistics can be viewed with `Statistics -> DNS -> General`
Under Service Stats, we can see a max response time of `512ms`

11. Query Pattern
Queried Domains (excluding normal traffic):
```
a1f3.data.exfil-test.net
9c02.data.exfil-test.net
77bd.data.exfil-test.net
e410.data.exfil-test.net
2ab6.data.exfil-test.net
f001.data.exfil-test.net
88c4.data.exfil-test.net
3d9e.data.exfil-test.net
6612.data.exfil-test.net
a0f5.data.exfil-test.net
bb31.data.exfil-test.net
12de.data.exfil-test.net
```
This looks a lot like someone's beaconing back data to a c2 server.

12. Beacon Domain
We see a single request to www.example.com, which I assume to be benign, and every other query is to `cdn-update-service.net`.
Counting these requests (not including the responses) we see `14 requests`.

13. Dropped File

First, lets filter by http request traffic by applying `http.requests` as our filter. 
Now, we see a bunch of GET requests to `cdn-update-service.net/beacon`. We also see a single request to `cdn-update-service.net/dl/svchost_update.exe`. 
We can export this object by going to `File -> Export Objects -> HTTP`. In the menu that pops up, we can see all the beacon files plus the .exe file, which Wireshark identifies as `application/octet-stream`

14. Something Extra
This is by far the hardest (and closest to a competition-level) challenge in the lab.
Looking at the beaconing to the malicious update service domain, I noticied a cookie called `src` with a (seemingly) random 3-byte value.

Keeping track of these as they were sent, we get
`Zmx hZ3 tjM DBr MWV fZH Ixc F8z eGY xbH 0=`

A few things stand out:
- Both uppercase and lowercase letters
- Numbers included with the letters
- An equals sign on the end

Since I've been doing challenges like this for a while, this matches up pretty well with what I know base64-encoded text looks like.

Using an online tool to decode the base64, we get the flag: `flag{c00k1e_dr1p_3xf1l}`

15. Incident Summary
The order of events in this packet capture are a little odd.
Usually, we see the malicious executable download happen first and then beaconing after that, to simulate a user (or bad actor) compromising a machine.
In this one, we see the beaconing already happening and the download last. This implies that the machine was already compromised by the time the capture was started, and that `svchost_update.exe` is likely a tool that the attacker needed/wanted to continue compromising the host/network.