# Installation and Configuration of Unbound on macOS:

### Install Unbound:
```
brew install unbound
```

### Configure Unbound

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
8. Verify Unbound service is running 

```
sudo brew services list

output:-

Name    Status  User File
unbound started root /Library/LaunchDaemons/homebrew.mxcl.unbound.plist
```

9. Configure macOS to Use Unbound:

- Open System Preferences > Network.
- Select your active network interface (e.g., Wi-Fi), click Advanced > DNS.
    - Add the following:
    - 127.0.0.1 (IPv4)
    - ::1 (IPv6)

10. Flush macOS DNS Cache:
```
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

### Testing the Setup


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