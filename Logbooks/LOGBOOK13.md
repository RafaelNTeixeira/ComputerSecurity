# Trabalho realizado na Semana #13 - Sniffing and Spoofing


# CTF13 - Find-my-TLS

1. We applied a filter to narrow down the displayed packets to TLS Client Hello messages:

```
tls.handshake.type == 1
```


2. We searched trough each one of the packets until we found the one that had the given random value (**52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27**) and retrieved the number of the frame.

![frame_start](Logbooks/img/Week13/foundPacketWithRandomNumber.png)

- frame_start = 814




3. By choosing the "Follow TCP Stream" option starting from the frame_start, we could access a chronological list of packets exchanged in the TCP connection associated with that specific frame. This feature allows you to observe the sequential flow of data within the connection, providing a detailed overview of the communication between the involved hosts. 

4.There, on the frame number 819, we found the "Finished" Message. Since it is the message that concludes the handshake, we can get the length of the TLS to determine the `size_of_encrypted_message` field of the flag, that is 80.

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
