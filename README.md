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

## Installation and Configuration on macOS:

#### Install Unbound:
```
brew install unbound
```

#### Configure Unbound

1. Locate the Configuration File: By default, the configuration file is:

```
/opt/homebrew/etc/unbound/unbound.conf
```

2. Open the Configuration File for Editing:

```
nano /opt/homebrew/etc/unbound/unbound.conf
```

3. Use the Following Configuration:

```
server:
    ##############################
    # General Settings
    ##############################
    verbosity: 1                   # Minimal logging (increase for debugging)
    interface: 127.0.0.1           # Listen on IPv4 loopback
    interface: ::1                 # Listen on IPv6 loopback
    port: 53                       # Default DNS port
    do-ip4: yes                    # Enable IPv4
    do-ip6: yes                    # Enable IPv6
    do-udp: yes                    # Enable UDP
    do-tcp: yes                    # Enable TCP

    ##############################
    # Security Settings
    ##############################
    root-hints: "/opt/homebrew/etc/unbound/root.hints"  # Location of root hints
    auto-trust-anchor-file: "/opt/homebrew/etc/unbound/root.key"  # DNSSEC root key
    val-permissive-mode: no       # Strict DNSSEC validation
    harden-glue: yes              # Harden against out-of-zone glue records
    harden-dnssec-stripped: yes   # Harden against DNSSEC stripping
    harden-below-nxdomain: yes    # Don't allow below NXDOMAIN results
    unwanted-reply-threshold: 10000000  # Rate limit unwanted replies
    hide-identity: yes            # Hide identity from queries
    hide-version: yes             # Hide version from queries
    use-syslog: no                # Disable syslog logging
    logfile: ""                   # No log file

    ##############################
    # Resource Control
    ##############################
    msg-cache-size: 4m            # DNS message cache size
    rrset-cache-size: 8m          # DNS record cache size
    cache-max-ttl: 86400          # Maximum cache time-to-live (24 hours)
    cache-min-ttl: 0              # Minimum cache time-to-live
    num-threads: 1                # Number of threads (tune for your CPU)

    ##############################
    # User and Chroot Settings (Disabled)
    ##############################
    chroot: ""                    # Disable chroot
    username: ""                  # Run as the current user (root)
    directory: "/opt/homebrew/etc/unbound"  # Working directory
    pidfile: "/opt/homebrew/etc/unbound/unbound.pid"

    ##############################
    # Forwarding with DoT (DNS-over-TLS)
    ##############################
    forward-zone:
        name: "."                  # Default zone for all queries
        forward-tls-upstream: yes  # Use DNS-over-TLS
        forward-addr: 1.1.1.1@853  # Cloudflare IPv4
        forward-addr: 2606:4700:4700::1111@853  # Cloudflare IPv6
        forward-addr: 8.8.8.8@853  # Google IPv4
        forward-addr: 2001:4860:4860::8888@853  # Google IPv6
```

4. Download Root Hints:
```
curl -o /opt/homebrew/etc/unbound/root.hints https://www.internic.net/domain/named.cache
```
5. Initialize DNSSEC Root Key:

```
unbound-anchor -a /opt/homebrew/etc/unbound/root.key
```

6. Check the Unbound Configuration File

```
unbound-checkconf /opt/homebrew/etc/unbound/unbound.conf
```

7. Start Unbound as a Service:

```
sudo brew services start unbound
```

8. Configure macOS to Use Unbound:

- Open System Preferences > Network.
- Select your active network interface (e.g., Wi-Fi), click Advanced > DNS.
    - Add the following:
    - 127.0.0.1 (IPv4)
    - ::1 (IPv6)

9. Flush macOS DNS Cache:
```
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

#### Testing the Setup


- Test DNS Resolution: IPv4
```
dig @127.0.0.1 example.com
```

- Test DNS Resolution: IPv6
```
dig @::1 example.com
```

- Test DNSSEC Validation:
```
dig +dnssec @127.0.0.1 dnssec-failed.org

dig @localhost example.nl +dnssec +multi
```
```
dig @localhost example.nl +dnssec +multi


; <<>> DiG 9.10.6 <<>> @localhost example.nl +dnssec +multi
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15602
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1232
;; QUESTION SECTION:
;example.nl.		IN A

;; ANSWER SECTION:
example.nl.		655 IN A 94.198.159.35
example.nl.		655 IN RRSIG A 13 2 3600 (
				20250114011743 20241231011157 21532 example.nl.
				rm4DUnGZtGT6xPqD4y3mJhpCQ9D30aPexZxuxUaJ2VmI
				L1KIlsuu4V9r+YrChdYC9RYu0y8PqPvagAsTOB1tjA== )

;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Sat Jan 04 19:21:41 IST 2025
;; MSG SIZE  rcvd: 161
```

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

#### I am adding the DNS traffic flow analysis [here](https://github.com/Diptiranjan9/Secure-DNS-for-macOS-using-Unbound/blob/main/Unbound-Traffic-Analysis.pcapng).

## Reference

- https://www.cloudflare.com/en-gb/learning/dns/what-is-dns/
- https://www.youtube.com/watch?v=ov73GnL66ww
- https://unbound.docs.nlnetlabs.nl/en/latest/getting-started/installation.html
- https://www.sidn.nl/en/modern-internet-standards/dnssec-validation-using-unbound-and-dnssec-trigger

