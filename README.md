# Tailscale-VPN-Installation-and-Secure-Remote-access-OMV-Homelab-
#Overview

The goal of this project was to securely access my home lab from anywhere without exposing services directly to the Internet or configuring port forwarding on my router.

My setup now allows me to securely access:

OpenMediaVault (OMV)
Nextcloud
My home LAN
Future services running on the network

using Tailscale.

#Creating the Tailnet

I created a Tailnet by visiting the Tailscale website and signing in with my Google account.

After creating the Tailnet, I added my Windows 11 desktop (knillpc1) as my first device.

I then installed the Tailscale app on my Samsung Galaxy S25 and signed into the same Tailnet.

Both devices received their own private Tailscale IP addresses.

To verify connectivity, I successfully pinged my phone from my Windows PC using its Tailscale IP.

#Installing Tailscale on OpenMediaVault

Using Command Prompt on my Windows PC, I connected to OMV using SSH.

ssh root@192.168.xxx.xxx

After logging in as root, I installed Tailscale.

curl -fsSL https://tailscale.com/install.sh | sh

Once the installation completed, I authenticated the NAS with my Tailnet.

tailscale up

After authenticating, the NAS appeared inside the Tailscale Admin Console as another device in my Tailnet.

Testing Remote Access

Using the newly assigned Tailscale IP address, I successfully connected to the OpenMediaVault web interface from my Windows PC.

http://100.xxx.xxx.xxx

This verified that the VPN was functioning correctly.

#Configuring Nextcloud

After verifying that I could successfully reach OpenMediaVault through its Tailscale IP address, I attempted to access my Nextcloud instance using:

http://100.xxx.xxx.xxx:8080

Instead of loading Nextcloud, I received an "Untrusted Domain" error.

This is a built-in security feature in Nextcloud.

When a browser connects to a website, it sends the address it used (known as the Host Header) to the server. Nextcloud compares that address against a list of approved addresses called Trusted Domains.

Since I was now accessing Nextcloud using its new Tailscale IP address, Nextcloud saw a request coming from an address it had never been configured to trust and intentionally refused the connection. This prevents attackers from tricking Nextcloud into serving requests through unauthorized domains or hostnames.

Because my Nextcloud installation is running inside a Docker container, I couldn't simply edit the configuration from the host operating system. Instead, I opened a shell inside the running container and used Nextcloud's built-in OCC (OwnCloud Console) command-line tool to modify the configuration.

Using OCC, I added the addresses that I wanted Nextcloud to trust:

localhost
the OMV LAN address (192.168.xxx.xxx)
the Tailscale address (100.xxx.xxx.xxx)

Each trusted domain was added individually to Nextcloud's configuration.

After updating the Trusted Domains list, I verified the changes and then reloaded the site.

Nextcloud now accepted connections from both:

http://192.168.xxx.xxx:8080

while on my local network, and

http://100.xxx.xxx.xxx:8080

when connected through Tailscale.

This step taught me how containerized applications maintain their own configuration, how Docker's exec command can be used to administer a running container, and why web applications often implement host validation as an additional security measure. It also reinforced the importance of understanding why a service is rejecting a connection rather than assuming the service itself is offline.

#Configuring the NAS as a Subnet Router

One of the primary goals of this project was to access my entire home network remotely.

To accomplish this, I configured OpenMediaVault as a Tailscale subnet router.

tailscale up --advertise-routes=192.168.xxx.xxx/24 --ssh

This advertises my home network to my Tailnet so authorized devices can securely reach LAN devices through the NAS.

Enabling IP Forwarding

Subnet routing requires Linux to forward traffic between interfaces.

I enabled IPv4 and IPv6 forwarding using:

cat <<EOF >/etc/sysctl.d/99-tailscale.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF

sysctl --system

After applying the configuration, the subnet route was approved inside the Tailscale Admin Console.

#Final Testing

To verify everything was working correctly:

Enabled Tailscale on my Galaxy S25
Disabled Wi-Fi
Switched to cellular data
Successfully connected to:
http://192.168.xxx.xxx

and

http://192.168.xxx.xxx:8080

The successful connection confirmed that subnet routing was working correctly and that my phone could securely access services hosted on my home network while away from home.

#Skills Demonstrated
Linux command-line administration
SSH remote management
Docker container administration
VPN deployment
Secure remote access
Private networking
Subnet routing
Network troubleshooting
Home lab infrastructure
OpenMediaVault administration
Nextcloud configuration
Cybersecurity best practices (avoiding exposed services and port forwarding)
