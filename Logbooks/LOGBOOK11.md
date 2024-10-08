# Trabalho realizado na Semana #11 - PKI Lab

## Task 1: Becoming a Certificate Authority (CA)

- What part of the certificate indicates this is a CA’s certificate?
```
X509v3 Basic Constraints: critical
    CA:TRUE
```

- What part of the certificate indicates this is a self-signed certificate?

```
Issuer: CN = www.modelCA.com, O = Model CA LTD., C = US
Subject: CN = www.modelCA.com, O = Model CA LTD., C = US

```

The fact that the issuer and the subject are the same, accuses it as a self assigned certificate.

- In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret
numbers p and q, such that n = pq. Please identify the values for these elements in your certificate
and key files.

```
e: 
    65537 (0x10001)

- - - - - - - - - - - - - - - - - - - - - - -
d:
    00:af:b7:cf:6a:1a:a8:e8:00:eb:4d:30:a2:af:e1:
    29:92:b5:c7:29:4c:59:48:4b:d7:29:74:6b:de:fa:
    37:83:f9:38:be:d7:9f:b7:da:aa:5e:ce:97:d8:57:
    f0:1e:49:16:d7:4a:fa:b7:3c:1d:41:69:87:3e:a7:
    7c:ab:22:f2:cc:e2:2c:5f:c5:9b:e7:7e:c5:3a:20:
    8d:9f:7e:2e:f4:cc:f4:17:ee:a2:c5:29:50:75:91:
    1a:d5:d6:9a:d0:78:7f:9c:0d:7b:2b:cf:a3:24:00:
    5e:73:32:a1:a3:1f:3b:96:e4:03:a1:56:5c:9f:28:
    10:e3:9b:67:80:1c:fb:ef:8f:42:3c:8d:ec:fa:63:
    2c:78:a4:67:a9:c9:64:7e:ec:44:f6:72:84:da:4e:
    49:17:c0:0e:78:39:4b:14:61:56:b7:34:7a:15:6d:
    90:fc:cf:41:5d:42:d6:f4:b4:d6:a3:98:3e:23:94:
    89:73:c5:a2:8e:66:e6:43:eb:5d:44:a1:66:f0:e2:
    17:36:22:8c:97:dc:50:54:31:b7:48:24:a9:e0:c4:
    ee:ab:a2:07:79:78:ae:f5:d9:0b:57:69:13:01:fd:
    7c:9d:ed:b9:24:79:33:3f:e4:95:5c:f1:d1:6e:c9:
    24:37:0b:82:33:96:86:a5:8e:3e:4c:59:58:b8:54:
    1c:32:8e:b6:6f:02:b6:92:0f:b8:fe:9c:42:35:08:
    f9:39:54:a0:83:ec:d3:bb:8a:6d:7b:91:82:dd:05:
    5a:67:e9:c8:be:cb:d9:ec:be:99:23:a1:b5:68:f5:
    09:39:6b:f6:1f:70:07:22:44:ea:38:d2:75:72:77:
    71:4f:b4:96:88:68:fe:c3:7a:1a:ee:c2:c6:a9:8c:
    94:6d:5a:a7:49:e8:8b:a8:71:64:a6:a4:ef:90:a8:
    de:66:17:6e:1c:f7:56:62:46:1c:16:91:9e:47:67:
    2d:1d:b3:b7:ff:33:49:2a:17:22:6d:8c:91:45:52:
    ec:ad:ae:38:33:da:45:2c:2d:8c:a5:41:97:b1:f7:
    ce:b2:a8:4e:54:4f:9d:2e:73:dd:95:39:9b:79:ff:
    d6:9b:92:b4:52:21:1c:ff:e8:27:e7:31:5e:65:bb:
    b1:81:bb:30:60:62:b0:33:db:95:a4:2d:1c:0f:93:
    83:bf:97:c5:df:66:03:e2:74:e2:3c:64:27:dc:47:
    b0:59:cc:6b:1e:68:a7:9c:4b:91:2f:ad:5e:51:ea:
    60:da:ff:1d:53:a0:bb:06:fe:da:7c:4d:a6:6b:cb:
    fb:06:7e:57:53:16:fc:c2:f6:61:27:4d:50:55:0d:
    57:d6:38:81:a4:9d:50:d2:14:c3:ac:1c:22:cd:42:
    6b:bf:6f

- - - - - - - - - - - - - - - - - - - - - - -

n:
    00:af:b7:cf:6a:1a:a8:e8:00:eb:4d:30:a2:af:e1:
    29:92:b5:c7:29:4c:59:48:4b:d7:29:74:6b:de:fa:
    37:83:f9:38:be:d7:9f:b7:da:aa:5e:ce:97:d8:57:
    f0:1e:49:16:d7:4a:fa:b7:3c:1d:41:69:87:3e:a7:
    7c:ab:22:f2:cc:e2:2c:5f:c5:9b:e7:7e:c5:3a:20:
    8d:9f:7e:2e:f4:cc:f4:17:ee:a2:c5:29:50:75:91:
    1a:d5:d6:9a:d0:78:7f:9c:0d:7b:2b:cf:a3:24:00:
    5e:73:32:a1:a3:1f:3b:96:e4:03:a1:56:5c:9f:28:
    10:e3:9b:67:80:1c:fb:ef:8f:42:3c:8d:ec:fa:63:
    2c:78:a4:67:a9:c9:64:7e:ec:44:f6:72:84:da:4e:
    49:17:c0:0e:78:39:4b:14:61:56:b7:34:7a:15:6d:
    90:fc:cf:41:5d:42:d6:f4:b4:d6:a3:98:3e:23:94:
    89:73:c5:a2:8e:66:e6:43:eb:5d:44:a1:66:f0:e2:
    17:36:22:8c:97:dc:50:54:31:b7:48:24:a9:e0:c4:
    ee:ab:a2:07:79:78:ae:f5:d9:0b:57:69:13:01:fd:
    7c:9d:ed:b9:24:79:33:3f:e4:95:5c:f1:d1:6e:c9:
    24:37:0b:82:33:96:86:a5:8e:3e:4c:59:58:b8:54:
    1c:32:8e:b6:6f:02:b6:92:0f:b8:fe:9c:42:35:08:
    f9:39:54:a0:83:ec:d3:bb:8a:6d:7b:91:82:dd:05:
    5a:67:e9:c8:be:cb:d9:ec:be:99:23:a1:b5:68:f5:
    09:39:6b:f6:1f:70:07:22:44:ea:38:d2:75:72:77:
    71:4f:b4:96:88:68:fe:c3:7a:1a:ee:c2:c6:a9:8c:
    94:6d:5a:a7:49:e8:8b:a8:71:64:a6:a4:ef:90:a8:
    de:66:17:6e:1c:f7:56:62:46:1c:16:91:9e:47:67:
    2d:1d:b3:b7:ff:33:49:2a:17:22:6d:8c:91:45:52:
    ec:ad:ae:38:33:da:45:2c:2d:8c:a5:41:97:b1:f7:
    ce:b2:a8:4e:54:4f:9d:2e:73:dd:95:39:9b:79:ff:
    d6:9b:92:b4:52:21:1c:ff:e8:27:e7:31:5e:65:bb:
    b1:81:bb:30:60:62:b0:33:db:95:a4:2d:1c:0f:93:
    83:bf:97:c5:df:66:03:e2:74:e2:3c:64:27:dc:47:
    b0:59:cc:6b:1e:68:a7:9c:4b:91:2f:ad:5e:51:ea:
    60:da:ff:1d:53:a0:bb:06:fe:da:7c:4d:a6:6b:cb:
    fb:06:7e:57:53:16:fc:c2:f6:61:27:4d:50:55:0d:
    57:d6:38:81:a4:9d:50:d2:14:c3:ac:1c:22:cd:42:
    6b:bf:6f

- - - - - - - - - - - - - - - - - - - - - - -

p:
    00:e9:fc:48:41:d9:b5:ee:e2:75:8a:c9:61:1b:a1:
    08:53:9a:22:09:ee:46:92:8d:07:6b:58:ad:c7:1a:
    6a:34:68:78:e0:6d:07:1e:f7:f0:16:9c:f9:58:a6:
    3b:80:79:9b:87:e8:de:7e:f1:c4:db:42:93:be:ab:
    38:fc:d5:61:65:a1:e0:89:8d:ad:ee:f3:46:64:cd:
    0a:a8:11:bb:bf:17:82:b2:a9:b1:50:e2:a2:66:7a:
    c6:f9:7d:5a:aa:28:f9:3b:2c:fd:69:2f:c7:08:91:
    2f:fd:0a:aa:e3:3c:9e:9f:00:ba:57:3c:9d:a4:32:
    e8:54:48:fe:6d:dc:c6:eb:31:77:98:61:89:51:da:
    39:0d:f7:7c:03:fd:04:20:b9:97:7d:77:0d:94:3b:
    44:cc:7d:7b:5f:09:8f:1b:b8:82:50:8c:90:ae:8e:
    a6:40:38:60:5a:fb:36:f8:a6:30:97:67:f6:c2:cb:
    69:ca:57:ea:84:da:3d:50:7f:c1:1c:59:13:92:1c:
    ac:58:b8:68:1c:1b:b1:9b:b8:1d:ea:f4:53:8c:20:
    01:e8:4c:98:03:c6:99:7a:e9:88:8e:98:13:08:be:
    1d:51:73:f8:f6:1a:06:51:54:e0:89:d2:4c:59:3b:
    7c:2a:83:3f:cb:72:f3:88:d3:0e:31:69:ea:48:34:
    98:a1

- - - - - - - - - - - - - - - - - - - - - - -

q:
    00:e9:fc:48:41:d9:b5:ee:e2:75:8a:c9:61:1b:a1:
    08:53:9a:22:09:ee:46:92:8d:07:6b:58:ad:c7:1a:
    6a:34:68:78:e0:6d:07:1e:f7:f0:16:9c:f9:58:a6:
    3b:80:79:9b:87:e8:de:7e:f1:c4:db:42:93:be:ab:
    38:fc:d5:61:65:a1:e0:89:8d:ad:ee:f3:46:64:cd:
    0a:a8:11:bb:bf:17:82:b2:a9:b1:50:e2:a2:66:7a:
    c6:f9:7d:5a:aa:28:f9:3b:2c:fd:69:2f:c7:08:91:
    2f:fd:0a:aa:e3:3c:9e:9f:00:ba:57:3c:9d:a4:32:
    e8:54:48:fe:6d:dc:c6:eb:31:77:98:61:89:51:da:
    39:0d:f7:7c:03:fd:04:20:b9:97:7d:77:0d:94:3b:
    44:cc:7d:7b:5f:09:8f:1b:b8:82:50:8c:90:ae:8e:
    a6:40:38:60:5a:fb:36:f8:a6:30:97:67:f6:c2:cb:
    69:ca:57:ea:84:da:3d:50:7f:c1:1c:59:13:92:1c:
    ac:58:b8:68:1c:1b:b1:9b:b8:1d:ea:f4:53:8c:20:
    01:e8:4c:98:03:c6:99:7a:e9:88:8e:98:13:08:be:
    1d:51:73:f8:f6:1a:06:51:54:e0:89:d2:4c:59:3b:
    7c:2a:83:3f:cb:72:f3:88:d3:0e:31:69:ea:48:34:
    98:a1

- - - - - - - - - - - - - - - - - - - - - - -
```

## Task 2: Generating a Certificate Request for Your Web Server

```
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" \
-passout pass:dees \
-addext "subjectAltName = DNS:www.bank32.com, \
DNS:www.bank32A.com, \
DNS:www.bank32B.com"
```

![BankCertificate](/Logbooks/img/Week11/PKI_-_task2.png)

## Task 3: Generating a Certificate for your server

In this lab, we will use our own trusted CA to generate
certificates

After running:

```
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

```
openssl x509 -in server.crt -text -noout
```

We can check all the distinct **Subjective Alternative Names** that were defined in the previous task:

```
X509v3 Subject Key Identifier: 
 66:2C:F2:84:7E:AF:C6:B7:2D:14:56:44:57:67:24:7B:3F:CF:ED:C9
X509v3 Authority Key Identifier: 
 keyid:A5:0A:84:53:EF:91:FA:13:77:4B:DA:50:94:82:79:88:A2:A9:3B:40
X509v3 Subject Alternative Name: 
 DNS:www.bank32.com, DNS:www.bank32A.com, DNS:www.bank32B.com
```

## Task 4: Deploying Certificate in an Apache-Based HTTPS Website

In this task, we will see how public-key certificates are used by websites to secure web browsing.

1. Setup the HTTPs website called Apache:

```
<VirtualHost *:443>
DocumentRoot /var/www/bank32
ServerName www.bank32.com
ServerAlias www.bank32A.com
ServerAlias www.bank32B.com
DirectoryIndex index.html
SSLEngine On
SSLCertificateFile /certs/bank32.crt 
SSLCertificateKeyFile /certs/bank32.key 
</VirtualHost>
```

2. Enable the Apache's module and enable this site:

```
$ a2enmod ssl
$ a2ensite bank32_apache_ssl
```

3. Start the server and gained access to it:
```
$ service apache2 start
```

4. As we started the server, we observed that it wasn't secure, but after adding the certificate, we can now have secure access:

![Secure](Logbooks/img/Week11/Secure.png)

## Task 5: Launching a Man-In-The-Middle Attack

In this task, we will show how PKI can defeat Man-In-The-Middle (MITM) attacks. 

1. We added the site `www.example.com` as a host:

``` 
$ cat /etc/hosts

10.9.0.80	www.bank32.com
10.9.0.80	www.example.com
``` 

2. With this, a warning was displayed, more specifically: `Warning: Potencial Security Risk Ahead`, since the domain `www.example.com` wasn't added to the list of the certificate when we inserted these commands in Task 2:

```
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" \
-passout pass:dees \
-addext "subjectAltName = DNS:www.bank32.com, \
DNS:www.bank32A.com, \
DNS:www.bank32B.com"
```


## Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA

For the last task, we assume that the root CA created in Task 1 is compromised by an attacker, and its private key
is stolen. Therefore, the attacker can generate any arbitrary certificate using this CA’s private key. With this, in this task, we will see the consequence of such a compromise.

1. First, we added `www.facebook.com` to the /etc/hosts folder:

```
[01/20/22]seed@VM:~/.../lab$ cat /etc/hosts

10.9.0.80	www.bank32.com
10.9.0.80	www.example.com
10.9.0.80	www.facebook.com
```

2. Then, we added `www.facebook.com` to the certificate DNS values:

``` 
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" \
-passout pass:dees -addext "subjectAltName = DNS:www.bank32.com, \
DNS:www.bank32A.com, \
DNS:www.bank32B.com, \
DNS:www.facebook.com"
``` 

3. We ran the following command:

``` 
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
``` 

4. The steps from Task 4 were repeated, this is, the certificates are copied to the container and added to the apache server configuration.

5. When entering the website again, we observed that the domain changed to `www.facebook.com`

![FacebookDomain](Logbooks/img/Week11/Facebook.png)

---


# CTF11 - RSA

1. Based on the Moodle form, we noticed that to solve this CTF, we would need to implement an algorithm to find prime numbers, preferably by using the Miller-Rabin algorithm. More specifically, we needed to discover the values of `p`, `q`, `e`, `n` and `d`:

    - **`p`**: prime number close to 2<sup>512</sup>
    - **`q`**: prime number close to 2<sup>513</sup>
    - **`e`**: public exponent
    - **`n`**: associated modulus
    - **`d`**:  private key obtained through `pow(e, -1, (p-1) * (q-1))`


**millerRabinAlgo():**

```py
def millerRabinAlgo(d, n):
    a = 2 + random.randint(1, n - 4)
    x = pow(a, d, n)
 
    if (x == 1 or x == n - 1):
        return True
 
    while (d != n - 1):
        x = (x * x) % n
        d *= 2
 
        if (x == 1):
            return False

        if (x == n - 1):
            return True
 
    return False # composite
```


2. We accessed the server and verified that the values `e`, `n`, and `ciphertext` were already provided.

3. To determine the remaining values, we started by searching all the possible prime numbers for `p` and `q`:

```py
def isPrime(n, k):
    if (n <= 1 or n == 4):
        return False

    if (n <= 3):
        return True
 
    d = n - 1
    while (d % 2 == 0):
        d //= 2
 
    for i in range(k):
        if (millerRabinAlgo(d, n) == False):
            return False
 
    return True

for i in range(primes_p - 1000, primes_p + 1000):
    if isPrime(i, 100): low_primes.append(i)  # 100 is used to increase the accuracy of the isPrime function

for i in range(primes_q - 1000, primes_q + 1000):
    if isPrime(i, 100): high_primes.append(i) # 100 is used to increase the accuracy of the isPrime function
```


4. We knew we had the correct values of `p` and `q` when the multiplication of a combination of possible values for `p` and `q` equaled the provided modulus `n`:

```py
for low in low_primes:
    for high in high_primes:
        if (low * high) == N:
            P = low
            Q = high
```


5. Having obtained the values of `p` and `q`, we determined the value of `phi` ((p-1) * (1-q)), to determine `d`:

```py
PHI = (P-1) * (Q-1)
d = pow(E, -1, PHI)
```


6. Obtaining the value of the private key, we decrypted it using the `dec()` function and printed its value, thus revealing the flag:

![flag](/Logbooks/img/Week11/flag.png)
