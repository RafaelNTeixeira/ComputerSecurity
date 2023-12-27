# Trabalho realizado na Semana #13 - Sniffing and Spoofing

This lab covers the following topics:

- How the sniffing and spoofing work
- Packet sniffing using the pcap library and Scapy
- Packet spoofing using raw socket and Scapy
- Manipulating packets using Scapy

## Lab Task Set 1: Using Scapy to Sniff and Spoof Packets

Many tools can be used to do sniffing and spoofing, but most of them only provide fixed functionalities. Scapy is different since it can be used not only as a tool, but also as a building block to construct other sniffing and spoofing tools.

### Task 1.1: Sniffing Packets

The objective of this task is to learn how to use Scapy to do packet sniffing in Python
programs.

On this task, this sample code was provided:

```py
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface=’br-c93733e9f913’, filter=’icmp’, prn=print_pkt)
```

#### Task 1.1A.

1. In the above program, for each captured packet, the callback function print `pkt()` will be invoked. This function prints out some of the information about the packet when we run the program with root privilege, however, if we run the program without it, the following error is shown:

```
Traceback (most recent call last):
  File "./sniffer.py", line 7, in <module>
    pkt = sniff(filter='icmp', prn=print_pkt)
  File "/usr/local/lib/python3.5/dist-packages/scapy/sendrecv.py", line 1036, in sniff
    sniffer._run(*args, **kwargs)
  File "/usr/local/lib/python3.5/dist-packages/scapy/sendrecv.py", line 907, in _run
    *arg, **karg)] = iface
  File "/usr/local/lib/python3.5/dist-packages/scapy/arch/linux.py", line 398, in __init__
    self.ins = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(type))  # noqa: E501
  File "/usr/lib/python3.5/socket.py", line 134, in __init__
    _socket.socket.__init__(self, family, type, proto, fileno)
PermissionError: [Errno 1] Operation not permitted
```

2. The `sniff()` function from the scapy package was included in the line highlighted to throw this issue. This error is thrown because sniffing requires raw access to network interfaces, which requires root privileges.

3. **These were the steps we followed:**
```
// Make the program executable
# chmod a+x sniffer.py

// Run the program with the root privilege
# sniffer.py

// Switch to the "seed" account, and run the program without the root privilege
# su seed
$ sniffer.py
```


#### Task 1.1B.

Usually, when we sniff packets, we are only interested certain types of packets. We can do that by setting filters in sniffing. 

Our objective was to demonstrate our sniffer program with the corresponding filters to:

- Capture only the ICMP packet
- Capture any TCP packet that comes from a particular IP and with a destination port number 23.
- Capture packets comes from or to go to a particular subnet. You can pick any subnet, such as
128.230.0.0/16; you should not pick the subnet that your VM is attached to.

- **Capture only the ICMP packet:**

1. We used the filter `icmp` and got this:

```
###[ Ethernet ]### 
    dst       = 08:00:27:08:74:bd
    src       = 08:00:27:34:8a:17
    type      = IPv4
###[ IP ]### 
    version   = 4
    ihl       = 5
    tos       = 0x0
    len       = 84
    id        = 47963
    flags     = 
    frag      = 0
    ttl       = 64
    proto     = icmp
    chksum    = 0xa745
    src       = 10.0.2.4
    dst       = 10.0.2.5
    \options   \
###[ ICMP ]### 
    type      = echo-reply
-...
```

- **Capture any TCP packet that comes from a particular IP and with a destination port number 23:**

1. We used the filter `tcp && src host 10.0.2.5 && dst port 23` and we got this:

```
###[ Ethernet ]### 
    dst       = 08:00:27:34:8a:17
    src       = 08:00:27:08:74:bd
    type      = IPv4
###[ IP ]### 
    version   = 4
    ihl       = 5
    tos       = 0x0
    len       = 60
    id        = 57259
    flags     = DF
    frag      = 0
    ttl       = 64
    proto     = tcp
    chksum    = 0x4308
...
```

- **Capture packets comes from or to go to a particular subnet. You can pick any subnet, such as 128.230.0.0/16; you should not pick the subnet that your VM is attached to:**

1. We used the filter `net 128.230.0.0/16` and got this:

```
###[ Ethernet ]### 
    dst       = 52:54:00:12:35:00
    src       = 08:00:27:34:8a:17
    type      = IPv4
###[ IP ]### 
    version   = 4
    ihl       = 5
    tos       = 0x0
    len       = 84
    id        = 30932
    flags     = DF
    frag      = 0
    ttl       = 64
    proto     = icmp
    chksum    = 0x34ea
    src       = 10.0.2.4
    dst       = 128.230.0.1
    \options   \
###[ ICMP ]### 
    type      = echo-request
    code      = 0
    chksum    = 0xc78c
    id        = 0xc8e
    seq       = 0x1
...
```


### Task 1.2: Spoofing ICMP Packets

As a packet spoofing tool, Scapy allows us to set the fields of IP packets to arbitrary values. The objective
of this task is to spoof IP packets with an arbitrary source IP address.

1. If we change the given code sample to the following, this is, by adding as IP source the value `10.0.2.3` and as IP destiny the value `10.0.2.5`:

```
>>> a = IP()
>>> a.src = '10.0.2.3'
>>> a.dst = '10.0.2.5'
>>> b = ICMP()
>>> p = a/b
>>> send(p)
```

2. The following packet is received by `10.0.2.5`:

```
###[ Ethernet ]### 
    dst       = 08:00:27:08:74:bd
    src       = 08:00:27:34:8a:17
    type      = IPv4
###[ IP ]### 
    version   = 4
    ihl       = 5
    tos       = 0x0
    len       = 28
    id        = 1
    flags     = 
    frag      = 0
    ttl       = 64
    proto     = icmp
    chksum    = 0x62d9
    src       = 10.0.2.3
    dst       = 10.0.2.5
    \options   \
###[ ICMP ]### 
    type      = echo-request
    code      = 0
    chksum    = 0xf7ff
    id        = 0x0
    seq       = 0x0
###[ Padding ]### 
    load      = '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```


### Task 1.3: Traceroute

The objective of this task is to use Scapy to estimate the distance, in terms of number of routers, between our VM and a selected destination. This is basically what is implemented by the `traceroute` tool but we will write our own.

- **Whats the process like?**

We send a packet to the destination, with its Time-To-Live `(a.ttl)` field set to 1. This packet will be dropped by the first router, which will send us an ICMP error message, telling us that the time-to-live has exceeded and will allow us to get the IP address of the first router.
We then increase our TTL field to 2, send out another packet, and get the IP address of the second router.
This process will be repeated until our packet reaches the destination. 


1. For that, we used a for cycle with a range of 100:

```py
from scapy.all import *

a = IP()
a.dst = '1.2.3.4'
b = ICMP()

for k in range(1, 100):
    a.ttl = k
    send(a/b)
```

2. Analysing the Wireshark logs, this is the traceroute that was taken:

`10.0.2.1` -> `10.12.0.1` -> `172.16.1.106` -> `172.16.1.210` -> `202.94.70.1` -> `203.116.245.177` -> `203.118.3.65` -> `203.118.6.25` -> `203.118.12.58` -> `203.118.15.214`


###  Task 1.4: Sniffing and-then Spoofing

In this task, we will combine the sniffing and spoofing techniques to implement the following sniff-and-then-spoof program and we will need two machines on the same LAN: the VM and the user container.

Our objective is to analyse and discuss what is observated when running the following pings:

- ping 1.2.3.4
- ping 10.9.0.99
- ping 8.8.8.8


1. **ping 1.2.3.4**
- We used the filter `icmp and host 1.2.3.4`.
- Only some packets are being sent and received. There is an estimation of 83.1% packet loss since the host doesn't exist on the Internet

![ping 1.2.3.4]()

2. **ping 10.9.0.99**
- We used the filter `icmp and host 10.9.0.99`
- When trying to send the packets, a warning `Destination Host Unreacheble` was given and there was 100% packet loss. This is due to the fact that since the host doesn't exist on the LAN, the source machine cannot obtain the MAC adress of the destination machine, preventing the transmission of the packets

![ping 10.9.0.99]()

3. **ping ping 8.8.8.8**
- We used the filter `icmp and host 8.8.8.8` 
- The packets are being sent and received successfully. There is 0% packet loss

![ping 8.8.8.8]()





# CTF13 - Find-my-TLS

1. We applied a filter to narrow down the displayed packets to TLS Client Hello messages:

```
tls.handshake.type == 1
```


2. We searched trough each one of the packets until we found the one that had the given random value (**52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27**) and retrieved the number of the frame.

![frame_start](Logbooks/img/Week13/foundPacketWithRandomNumber.png)

- frame_start = 814




3. By choosing the "Follow TCP Stream" option starting from the frame_start, we could access a chronological list of packets exchanged in the TCP connection associated with that specific frame. This feature allows you to observe the sequential flow of data within the connection, providing a detailed overview of the communication between the involved hosts. 

4. There, on the frame number 819, we found the "Finished" Message. Since it is the message that concludes the handshake, we can get the length of the TLS to determine the `size_of_encrypted_message` field of the flag, that is 80.

![frame_end_AND_size_encrypted_message](Logbooks/img/Week13/frame_end_AND_size_of_encrypted_message.png)

- frame_end = 819
- size_of_encrypted_message = 80


5. On the frame_start, we searched for the field Cipher Suites. It contained 2 values, however the one that we wanted was the one that promoted data exchange encryptation between the TLS conexion, namely the RSA.

![cipher_suite](Logbooks/img/Week13/cipherSuit.png)

- cipher_suite = TLS_RSA_WITH_AES_128_CBC_SHA256



6. To obtain the field `total_encrypted_appdata_exchanged` we made the sum of all the TLS lengths of the packets with the type Application Data.

![ApplicadtionData1](Logbooks/img/Week13/ApplicationData1.png)

![ApplicadtionData2](Logbooks/img/Week13/ApplicationData2.png)


- total_encrypted_appdata_exchanged = 80 + 1184 = 1264

---


With this, the flag that we constructed ended up like this:

`flag{814-819-TLS_RSA_WITH_AES_128_CBC_SHA256-1264-80}`
