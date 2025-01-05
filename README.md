# Unbound as a Local DNS Stub Resolver with DNSSEC and DoT (Single Machine)

## DNSSEC and DoT:

### DNSSEC

#### Purpose:

- `DNSSEC` ensures the `integrity and authenticity` of DNS data to prevent attackers from tampering with it (e.g., DNS spoofing or cache poisoning).

#### How It Works:

- Uses cryptographic signatures to verify that the DNS data (like IP addresses) comes from an authoritative source and hasn’t been altered in transit.
- DNS records are signed with a private key, and resolvers validate these records using a corresponding public key.

#### Protection Provided:

- Prevents attackers from injecting malicious or incorrect DNS responses.
Ensures the authenticity of DNS data.

#### Limitations:

- DNS queries and responses are still sent in plaintext, so they can be observed by third parties (e.g., ISPs or attackers).

### DoT (DNS over TLS)

#### Purpose:

- `DoT` ensures the `confidentiality and privacy` of DNS queries by encrypting them.

#### How It Works:

- DNS traffic is sent over a secure TLS connection (similar to HTTPS encryption for websites).
- Protects the DNS queries from being intercepted or monitored by third parties.

#### Protection Provided:

- Encrypts DNS queries and responses, preventing eavesdropping or man-in-the-middle attacks.
- Hides the content of DNS queries (e.g., the domain name being queried).

#### Limitations:

- Does not verify the authenticity of DNS data (e.g., it doesn’t prevent receiving forged DNS responses unless used with DNSSEC).

### Combining DNSSEC and DoT for Enhanced Security:

To mitigate the limitations of both technologies, it is highly recommended to use `DNSSEC and DoT` together.

- DNSSEC ensures the authenticity of DNS data, while DoT provides confidentiality and privacy.
- By combining them, you create a robust security layer that protects against both data tampering and eavesdropping.

## Why Unbound Stands Out:

`Unbound` is a popular DNS resolver known for its:

- ***Strong Security***: Excellent DNSSEC validation and DoT support for privacy.

- ***Performance***: Lightweight and efficient, ideal for various environments.

- ***Flexibility***: Highly configurable to suit specific needs.

- ***Open Source***: Active community, continuous development, and community support.

Compared to BIND, Unbound is often considered more lightweight and resource-efficient, with a stronger focus on security. It's a solid choice for those prioritizing security and performance.

### Verification command output:

```
dig @localhost -4 google.com

; <<>> DiG 9.10.6 <<>> @localhost -4 google.com
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12626
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		190	IN	A	142.250.193.46

;; Query time: 649 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sat Jan 04 19:28:23 IST 2025
;; MSG SIZE  rcvd: 55
```

```
dig @localhost -6 google.com AAAA

; <<>> DiG 9.10.6 <<>> @localhost -6 google.com AAAA
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24494
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.			IN	AAAA

;; ANSWER SECTION:
google.com.		290	IN	AAAA	2404:6800:4009:825::200e

;; Query time: 406 msec
;; SERVER: ::ffff:127.0.0.1#53(127.0.0.1)
;; WHEN: Sat Jan 04 19:28:51 IST 2025
;; MSG SIZE  rcvd: 67
```

## Cybersecurity Advantages of Using Unbound with DNSSEC and DoT

#### Protection Against DNS Attacks:

- DNSSEC prevents spoofing and cache poisoning by validating DNS responses with cryptographic signatures.

#### Privacy via Encryption:

- DNS-over-TLS (DoT) encrypts DNS traffic, preventing ISPs and attackers from spying on or tampering with your DNS queries.

#### Local Control:

- Running a local resolver ensures your DNS data isn't logged or controlled by third-party servers.

#### Defense Against MITM Attacks:

- Secure communication with upstream DNS servers thwarts man-in-the-middle (MITM) attacks.

#### Reduced Attack Surface:

- Binding Unbound to `127.0.0.1` and `::1` prevents external access and blocks amplification attack vectors.

#### System Hardening:

- No logging reduces sensitive data exposure, and strict DNSSEC validation ensures data integrity.

### Traffic Analysis:

#### I am adding the DNS traffic flow pcapfile for both IPv4 and IPv6 [here](https://github.com/Diptiranjan9/Secure-DNS-for-macOS-using-Unbound/tree/main/pcapfile).

## Reference

- https://www.cloudflare.com/en-gb/learning/dns/what-is-dns/
- https://www.youtube.com/watch?v=ov73GnL66ww
- https://unbound.docs.nlnetlabs.nl/en/latest/getting-started/installation.html
- https://www.sidn.nl/en/modern-internet-standards/dnssec-validation-using-unbound-and-dnssec-trigger

