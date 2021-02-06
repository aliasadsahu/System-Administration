Setup OpenVPN Server on Pfsense
pfsense
Overview#

Let's setup OpenVPN server on Pfsense firewall, so that we can connect a client device on the road.
Static IP vs Dynamic IP#
note

I'm assuming that the IP you have from your ISP won't change. If your ISP changes your IP, you'll have to tweak the configuration file for every new IP change. Or you could add a dynamic DNS server in order to solve this problem.
Install OpenVPN package on Pfsense#

Pfsense, system, package manager, available packages, search openvpn, select install
Setup OpenVPN server#

Pfsense, VPN, OpenVPN, select Wizards tab

We'll do a simple installation, using a local user that we'll created in Pfsense later on.
Step 1: Authentication#

    Type of Server: Select Local User Account
    Next

Step 2: Add Certificate Authority#

    Descriptive Name: Home_OpenVPN
    Next

Step 3: Add Server Certificate#

    Descriptive Name: Home_OpenVPN_cert
    Next

Step 4: Setup Server#

    General OpenVPN Server Information:
        Local Port: 1194
            By default, the OpenVPN server uses port 1194 and the UDP protocol to accept client connections. If you need to use a different port because of restrictive network environments that your clients might be in, you can change the port option. If you are not hosting web content your OpenVPN server, port 443 is a popular choice since this is usually allowed through firewall rules.
        Description: Home_OpenVPN_Instance

    Cryptographic Settings:
        Hardware Crypto: Change from default No Hardware Crypto Available to your preference, if you have a hardware capable of doing this.

    Tunnel Settings:
        Tunnel Network: 192.168.10.0/24
            Should be a new, unique network that does not exist anywhere in the current network or routing table.
        Redirect Gateway:
            Turn off if you want your client to reach the server's LAN IPs and servers, but not the actual website traffic.
                This is called split tunnel VPN.
                Use this when you need to access a server at home/work for example.
            Turn On if you want to have everything you're doing go out through the server's WAN.
                This is called full tunnel VPN.
                Use this where you don't trust the network, such as coffee shop or hotel.
        Local Network: 192.168.1.0/24 (if you wish to allow traffic from VPN to your local LAN network, enter it here.
            If you don't want to give access to your LAN network, leave it blank.
            If you have multiple network, list them like 192.168.1.0/24, 192.168.2.0/24
        Concurrent Connections: - Leave a blank to allow unlimited number of clients to connect at once.
            If you have a specific requirement, you might need to restrict this to a certain number.

    Client Settings:

        DNS Default Domain:
            This is the domain.
            I used mylocal (from Pfsense, system, general setup, domain)
            If you enable this, the VPN clients that connect to your LAN, will be able to ping hostnames.
            If you don't enable this, the VPN clients that connect to your LAN, don't be able to ping hostnames, but can ping IPs only.

        DNS Server 1:
            Set to public DNS server if you wish to have an unfiltered DNS traffic.
                1.1.1.1 for Cloudflare
                8.8.8.8 for Google
                9.9.9.9 for Quad9
            Set to private DNS server if you wish to filter your DNS traffic through Pihole.
                192.168.2.2 (I used my Pihole DNS server)
                Private_IP (You could use an Active Directory DNS server)

        NetBIOS Options:
            If you want to have the ability for your client to access to Windows shares, enable this.
            If you don't want this, keep it disabled.
            To test this fully...

    Next

Step 5: Firewall Rules#

If you're setting up OpenVPN for the first time (if you're setting up a 2nd VPN server, you won't need to enable these again)

    Enable Firewall Rule
        Permits connections to this OpenVPN server process from clients anywhere on the internet.
    Enable OpenVPN Rule
        Allows all traffic from connected clients to pass inside the VPN tunnel.
    Next

Step 6: Wizard Completed#

    Next

Server Mode explained#

The OpenVPN Server Mode allows selecting a choice between requiring Certificates, User Authentication, or both. The wizard defaults to Remote Access (SSL/TLS + User Auth). The possible values for this choice and their advantages are:

    Remote Access (SSL/TLS + User Auth):
        Requires both certificates AND username/password
        Each user has a unique client configuration that includes their personal certificate and key.
        Most secure as there are multiple factors of authentication (TLS Key and Certificate that the user has, and the username/password they know)

    Remote Access (SSL/TLS):
        Certificates only, no auth
        Each user has a unique client configuration that includes their personal certificate and key.
        Useful if clients should not be prompted to enter a username and password
        Less secure as it relies only on something the user has (TLS key and certificate)

    Remote Access (User Auth):
        Authentication only, no certificates
        Useful if the clients should not have individual certificates
        Commonly used for external authentication (RADIUS, LDAP)
        All clients can use the same exported client configuration and/or software package
        Less secure as it relies on a shared TLS key plus only something the user knows (Username/password)

Step 7: Edit the Server#

Pfsense, VPN, OpenVPN, select Servers tab, edit

    Select Remote Access (User Auth)
        For simplicity, I'll use this to demo.
        Your use case may be different.

Step 8: Add a User#

    The reason we need to create a new account, is we don't want to use our admin account to a VPN user.
    Pfsense, System, User Manager, Select Add
        Username: vpnuser
        Password: xxx (you'll need to use this password on the client later on) (I suggest a randomly generated 64 characters and symbols)
        OPTIONAL: If you selected Remote access (SSL/TLS) or Remote access (SSL/TLS + User Auth), then click on Add user certificate, name it.
        Save

Step 9: Download the configuration file#

    For Windows, click on the Most Clients (under Inline configurations)
    For Mac, click on the Most Clients (under Inline configurations)
    For Linux, click on the Most Clients (under Inline configurations)
    For Google Android, click on the OpenVPN Connect (under Inline configurations)
    For Apple iOS, click on the OpenVPN Connect (under Inline configurations)

note

I didn't download the installation files (for Windows, Mac, etc) from this page, as these look outdated. Just install the latest version of OpenVPN installation file directly from the website.
Optional: View contents of this configuration file#

For those who want to look at this configuration file, open the file.ovpn with a text editor.
dev tun
persist-tun
persist-key
cipher AES-128-CBC
ncp-ciphers AES-128-GCM
auth SHA256
tls-client
client
resolv-retry infinite
remote SERVER_PUBLIC_IP 1194 udp4
auth-user-pass
remote-cert-tls server

<ca>
-----BEGIN CERTIFICATE-----
aaaaaaaaaaaaaaaaaaaaaaaaaaa
-----END CERTIFICATE-----
</ca>
setenv CLIENT_CERT 0
key-direction 1
<tls-auth>
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
bbbbbbbbbbbbbbbbbbbbbbbbbbbbb
-----END OpenVPN Static key V1-----
</tls-auth>

I've blanked out:

    My public IP and replaced with SERVER_PUBLIC_IP
    2 certificates, replaced with aaa and bbb

Setup OpenVPN on Apple iOS or Android#

    Download link for iOS.
    Download link for Android.
    Install and import the previously downloaded file (called pfSense-UDP4-1194-ios-config.opvn or similar)

Setup OpenVPN on MacOS or Windows or Linux#

    Download link for MacOS or Windows or Linux.
    From Pfsense, VPN, OpenVPN, Client Export, I selected the Most Clients (under Inline configurations), which will download name.opvn
    Import this .ovpn file into OpenVPN.
    Enter the username vpnuser and password you setup in step 8 above.

Testing Your VPN Connection#

    Disable OpenVPN, open a browser and go to dnsleaktest.com
        The site will return the IP address assigned by your internet service provider and as you appear to the rest of the world. To check your DNS settings through the same website, click on Extended Test and it will tell you which DNS servers you are using.
    Enable OpenVPN and refresh the browser.
        A completely different IP address (that of your VPN server) should now appear, and this is how you appear to the world. Again, DNSLeakTest’s Extended Test will check your DNS settings and confirm you are now using the DNS resolvers pushed by your VPN.

Optional: Add widget to Pfsense dashboard#

    Pfsense, Dashboard, Click on plus icon, click on OpenVPN, drag it into place, click on save icon.

    Connect a client to OpenVPN server, and as soon as you do, you'll see the client come up in Pfsense dashboard. It will show the public IP from the client device and the IP 192.168.10.2.

    You can also see additional information from Pfsense, Status, OpenVPN.

Test #1: OpenVPN over ISP's Fiber Optic Cable#
note

Take the tests with a grain of salt, as your conditions will vary.

Tests done with speedtest.net:

    Home Fiber (over LAN, No VPN)
        Ping: 3 ms
        Download: 177 mbps
        Upload: 177 mbps
    Work Fiber (over LAN, No VPN)
        Ping: 2 ms
        Download: 511 mbps
        Upload: 510 mbps
    Connected to Home using Work's LAN (using OpenVPN) using Desktop
        Ping: 3 ms
        Download: 107 mbps
        Upload: 100 mbps
    Connected to Home using Work's Wifi (using OpenVPN) using iPhone
        Ping: 8 ms
        Download: 130 mbps
        Upload: 120 mbps
    Connected to Home using Work's Wifi (using OpenVPN) using Macbook
        Ping: 5 ms
        Download: 141 mbps
        Upload: 115 mbps

Conclusion:

    At home, I have 3 ms ping. At work, I have 2 ms ping. When I connected to my Home's connection over OpenVPN from work, I'm getting 2 ms ping.
        Incredible!

Test #2: OpenVPN over Cellular Provider's LTE Connection#
note

Take the tests with a grain of salt, as your conditions will vary.

Tests done with speedtest.net:

    On iPhone, using LTE, without VPN
        Ping 20 ms
        Download 27 mbps
        Upload 7 mbps
    On iPhone, Using LTE, with OpenVPN, connected to Home (which has a Fiber optic connection, with ping of 3 ms and 177 mbps down/up)
        Ping 77 ms
        Download 40 mbps
        Upload 10 mbps

Optional: In order to increase your OpenVPN reliability, we can add another instance#

Remember, when I said that running OpenVPN over port 1194 UDP might be blocked by some firewalls? Well, we can run a 2nd instance of OpenVPN over port 443 TCP as a backup, in case the first instance get's blocked for whatever reason.

How to do that?

    Run Wizard again, and change the following:

    Protocol, instead of UDP on IPv4 only use TCP on IPv4 only

    Local Port, instead of 1194 use 443

    Description, instead of Home_OpenVPN_Instance use Home_OpenVPN_Instance_2

    Tunnel Network: 192.168.11.0/24 (instead of 192.168.10.0/24 like I used for first instance) (this makes it easy to see when you're being routed via port 443 TCP)

    Firewall Rule turn on.

    OpenVPN rule turn off. (since it will match your previously created rule, thereby creating a duplicate entry for no reason)

    Finish

    When you go to Pfsense, VPN, OpenVPN, you'll see the 2 instances.

    Now, you'll have to repeat the Client Export process:
        This time, select the Remote Access Server as Home_OpenVPN_Instance_2 TCP4:443
    Import into clients again
    Enable OpenVPN on the client and test the connection. Everything should be working the same.