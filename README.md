ssrfsocks: SOCKS-based SSRF tunneling proxy

  ssrfsocks utilizes an existing Server-Side Request Forgercy vulnerability to
  tunnel traffic through it. This code acts as a SOCKS4 proxy server, and
  relays traffic through the SSRF vuln and returns the result. This concept is
  based on the Blackhat 2012 presentation by Polyakov & Chastukhin on SSRF and
  XXE tunneling.

Limitations:

  Due to the nature of SSRF vulnerabilities, only one response is made from a
  request. This means that it is not possible to run an interactive session. We
  can however send single-shot messages to internal services. This means it's 
  possible to, for example, send an email via SMTP, and grab the banner of most
  services. This will tend to be most useful with simple protocols like SMTP 
  and HTTP, and ineffective with protocols such as SSH and TLS.

Example:

    $ ./ssrfproxy "http://victim.com/openurl?url="
    listener ready on port 65432

    $ echo -e "HELO localhost\nMAIL FROM:root@victim.com\nRCPT TO: root@ultramegaman.com\nDATA\nSubject: test\n\nthis is a test\n.\nQUIT\n" \
      | proxychains nc 127.0.0.1 25

  In the above scenario, it's necessary to configure proxychains.conf:
  
    [ProxyList]
    socks4 127.0.0.1 65432

Testing:

  Currently, the SOCKS4A code is untested, as proxychains doesn't seem to
  support it. I found limited success with running nmap over this proxy, as 
  nmap reports closed ports as open, and some open ports as closed.
  
  This script was tested against a vulnerable app that passed a user-controlled
  value to code that treated the value as a URI, fetched it, and returned the
  result. It's likely not usable against XXE vulnerabilities without some
  modification.
