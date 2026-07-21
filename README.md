# Tailscale-VPN-Installation-and-Secure-Remote-access-OMV-Homelab-
Overview

The goal of this project was to securely access my home lab from anywhere without exposing services directly to the Internet or configuring port forwarding on my router.

My setup now allows me to securely access:

OpenMediaVault (OMV)
Nextcloud
My home LAN
Future services running on the network

using Tailscale.

Creating the Tailnet

I created a Tailnet by visiting the Tailscale website and signing in with my Google account.

After creating the Tailnet, I added my Windows 11 desktop (knillpc1) as my first device.

I then installed the Tailscale app on my Samsung Galaxy S25 and signed into the same Tailnet.

Both devices received their own private Tailscale IP addresses.

To verify connectivity, I successfully pinged my phone from my Windows PC using its Tailscale IP.

Installing Tailscale on OpenMediaVault

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

Configuring Nextcloud

Nextcloud initially displayed an "Untrusted Domain" error when accessed through its Tailscale IP.

This occurred because Nextcloud only accepts connections from addresses listed in its Trusted Domains configuration.

Using the Docker OCC utility inside the Nextcloud container, I added:

localhost
the OMV LAN IP
the Tailscale IP

to the Trusted Domains list.

After updating the configuration, Nextcloud was accessible remotely.

http://100.xxx.xxx.xxx:8080
Enabling Tailscale SSH

I enabled Tailscale SSH by running:

tailscale up --ssh

This allows SSH connections to be made securely over the encrypted Tailnet without exposing SSH directly to the Internet.

Configuring the NAS as a Subnet Router

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

Final Testing

To verify everything was working correctly:

Enabled Tailscale on my Galaxy S25
Disabled Wi-Fi
Switched to cellular data
Successfully connected to:
http://192.168.xxx.xxx

and

http://192.168.xxx.xxx:8080

The successful connection confirmed that subnet routing was working correctly and that my phone could securely access services hosted on my home network while away from home.

Skills Demonstrated
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
