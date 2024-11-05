Overview of Tunneling
    Tunneling - Tunneling encapsulates a protocol inside another protocol.
        Encapsulation
        Transmission
        Decapsulation

Overview of Tunneling
    Tunneling used in the IPv6 Transition
        IPv6 over IPv4
        Dual Stack
        6in4
        6to4
        4in6
        Teredo
        ISATAP
Covert Channels
    Using common and legitimate protocols to transfer data in illegitimate ways.
    Unauthorized/hidden communication between entities.
    Utilizes computer system resources, mechanisms, or protocols.
    Transmits information contrary to design intent.
    Bypasses security, violates policies, leaks sensitive data.
Common Protocols used with Covert Channels
    ICMP
    ICMP works with one request and one reply answer
        Type 8 code 0 request
        Type 0 code 0 answer
    Check for:
        Payload imbalance
        Request/responce imbalance
        Large payloads in response
  DNS
    DNS is a request/response protocol
    1 request typically gets 1 response
    Payloads generally do no exceed 512 bytes
    Check for:
        Request/response imbalances
        Unusual payloads
        Burstiness or continuous use
  HTTP
     Request/Response protocol to pull web content
    GET request may include .png, .exe, .(anything) files
    Can vary in sizes of payloads
    Typically "bursty" but not steady
    
Steganography
    Hiding messages inside legitimate information objects
        Methods:
            --Injection--
    Done by inserting message into the unused (whitespace) of the file, usually in a graphic
    Second most common method
    Adds size to the file
    Hard to detect unless you have original file
    tools:
        StegHide
        --Substitution--
    Done by inserting message into the insignificant portion of the file
    Most common method used
    Elements within a digital medium are replaced with hidden information
    Example
        Change color pallate (+1/-1)
          --Propagation--
    Generates a new file entirely
    Needs special software to manipulate file
        tools:
            StegSecret
            HyDEn
            Spammimic
            
--Secure Shell (SSH)
SSH
    Various Implementations (v1 and v2)
    Provides authentication, encryption, and integrity.
    Allows remote terminal sessions
    Can enable X11 Forwarding
    Used for tunneling and port forwarding
    Proxy connections
--SSH Architecture
    Client vs Server vs Session
    Keys:
        User Key - Asymmetric public key used to identify the user to the server
        Host Key - Asymmetric public key used to identify the server to the user
        Session Key - Symmetric key created by the client and server to protect the session’s communication.
--Configuration Files
    Client Configuration File (/etc/ssh/ssh_config)
    Server Configuration File (/etc/ssh/sshd_config)
    Known Hosts File (~/.ssh/known_hosts)
--SSH Port Forwarding
    Creates channels using SSH-CONN protocol
    Allows for tunneling of other services through SSH
    Provides insecure services encryption
--
ssh student@172.16.82.106
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:RO05vd7h1qmMmBum2IPgR8laxrkKmgPxuXPzMpfviNQ.
Please contact your system administrator.
Add correct host key in /home/student/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/student/.ssh/known_hosts:1
remove with:
ssh-keygen -f "/home/student/.ssh/known_hosts" -R "172.16.82.106"
ECDSA host key for 172.16.82.106 has changed and you have requested strict checking.
Host key verification failed.

--SSH Options
    -X - X11 forwarding
    -L - Creates a port on the client mapped to a ip:port via the server
    -D - Creates a port on the client and sets up a SOCKS4 proxy tunnel where the target ip:port is specified dynamically
    -R - Creates the port on the server mapped to a ip:port via the client
    -NT - Do not execute a remote command and disable pseudo-tty (will hang window)
--------------------------
--Local Port Forwarding--
ssh -p <optional alt port> <user>@<server ip> -L <local bind port>:<tgt ip>:<tgt port> -NT
or
ssh -L <local bind port>:<tgt ip>:<tgt port> -p <alt port> <user>@<server ip> -NT


--Local Port Forwarding -- SSgt Woods edition
  ssh <user><ip> -L <bind_port>:<tgt>:<tgt_port> (-p = alternate port)
                    ^^opening on your machine  

IH> nmap -sV -T4 -p20-23,80 BH1
    -- gets back open ports and versions
IH> ssh user@BH1 -L RHP1:127.0.0.1:80 -NT
    -- map ports used
IH> nc 127.0.0.1 RHP1
    -- get http banner
IH> nc 127.0.0.1:RHP1 -r
    -- gets contents of the webserver
IH> user@BH1
    -- gets on BH1 for host enumeration
BH1> hostname, whoami, ip addr, ip neigh, uname -a, ip route, ss -ntlp, ping sweep, cat /etc/passwd
      --host enumeration, found internal ip, closes connection
  -
--Dynamic Port Forwarding--
Proxychains default port is 9050
    Creates a dynamic socks4 proxy that interacts alone, or with a previously established remote or local port forward.
    Allows the use of scripts and other userspace programs through the tunnel.
--
IH> ssh user@BH1 -D 9050 -NT
     --map ports 9050 ->BH1 -> *
IH> proxychains nmap -T4 -vvvv -p21-23,80 <int IP> -Pn
IH> proxychains nc 127.0.0.1 21
IH> proxychains wget -r ftp://127.0.0.1
      -- only if anonymous is allowed
IH> proxychains ftp 127.0.0.1
      -- needs credentials
  ftp> passive
      -- have to switch to passive mode
        --run commands, ls , cd (grab files)
**Make sure to check mappings**
once done with ftp time to ssh to the <int IP>
IH> ssh user@BH1 -L 52201:<int IP>:22 -NT
 **add to mapping of ports**
IH> nc 127.0.0.1 52201
    -get ssh version
IH> ssh user@127.0.001 -p52201
    --Host enumeration "hostname BPH1"
BPH1> Host enumeration hostname, whoami, ip addr, ip neigh, uname -a, ip route, ss -ntlp, ping sweep, cat /etc/passwd
      -- no more interfaces or hosts
BPH1> ssh user@BH1int -R 52299:127.0.0.1:80 -NT
      --opens port 52299 on BH1
IH> ssh user@BH1 -L 52202:127.0.0.1:52299 -NT
      **add to mapping ports**
IH> nc 127.0.0.1 52202
IH> wget -r 127.0.0.1:52202
    --gets web server info of BPH1
IH> ssh user@127.0.0.1 -p52201 -D 9050 -NT
 **update map**
 IH> proxychains wget -r ftp://127.0.0.1 or ftp 127.0.0.1
 -
--**______________________________________________________________**--

Local Forwarding
<host>~$**<user>@<IP>** -L <Bind_Port>:**<Tgt_IP>:<Tgt_Port>** -NT

Remote Forwarding
<host>~$**<user>@<IP>** -R **<Bind_Port>**:<Tgt_IP>:<Tgt_Port> -NT

Proxychains
<host>~$**<user>@<IP>** -D 9050 -NT
                    **-p<port> "specify RHP"**
                    
**System your authenticating to** = "**"

"scp using port forwards"
scp **<RHP1>** **<user>@<IP>**:secrets/notsecret Recon/
                                ^^what u want       ^^where u want it






7. B
8. D
9. C
10. B
11. D
12. A
13. B
14. A
15. C
16. A
17. D
18. B
19. C
20. 


















































































