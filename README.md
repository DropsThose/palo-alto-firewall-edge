# Palo Alto Firewall VM Edge Project - IN PROGRESS
## Overview

In this project, I put my entire home network behind a Palo Alto firewall VM. The purpose is to add the pressure of impacting more than just myself: if I break something in the config, it's not just me who is affected, but my family, which is pressure you can't replicate in a typical lab environment. The goal is to take on the responsibility of managing my home network's firewall, practicing working under pressure and with urgency to solve problems that now affect real users.

### Technology Utilized
- Palo Alto Firewall VM (11.2.12)
- Cisco WS-C3750X-24T-S (15.2(4)E10)
- Proxmox VE (9.2.5)

## Baseline Setup
### Cisco VLANs/interfaces
<img src="/assets/Cisco vlans.png" />
On my switch, I created VLAN 60 to use for my home network. 
<img src="/assets/Cisco interfaces.png" />
I added various wired devices that used to be plugged into my home's Comcast router to my switch as access ports to VLAN 60. 

### Comcast router bridge mode
<img src="/assets/Comcast bridge mode.png" />

I put my home Comcast router into bridge mode to get its WAN IP onto my PA VM's WAN link. Putting the router into bridge mode disabled its Wi-Fi broadcasting, so I had to use another access point to connect wireless devices to my home VLAN. 

On my other access point, I copied the SSID and password that the Comcast router had previously broadcast, allowing wireless devices to seamlessly transition to my new VLAN 60! 
### Proxmox setup
<img src="/assets/Proxmox PA VM Hardware.png" />
In the Proxmox hardware settings for my PA VM, I configured network devices to allow the PA VM's virtual interfaces to connect to the physical interfaces on my T710 Proxmox server. net0 is my management interface, which I have segmented to VLAN 10. Net1 is the bridge port on my Comcast router. Net2 is connected to my trunk port to carry all my VLAN traffic from the Cisco to the T710. Net1 maps to ethernet1/1 on the PA VM side; net2 maps to ethernet1/2, etc. 

### PA Firewall configuration
<img src="/assets/PA Interfaces.png" />

I created subinterfaces under ethernet1/2 for all of my Cisco VLANs. I also created zones for each interface and subinterface. For ethernet1/1, my WAN link, in other words, the dangerous internet, I gave it the zone "untrusted". For the subinterfaces, I created zones that matched the names of my Cisco VLANs. 

I assigned the subinterface for my home network's VLAN 60 to the "home" zone.
<img src="/assets/PA DHCP Servers.png" />
I created a DHCP server and assigned it to my home subinterface. The DHCP server will allow devices connected to my home network to automatically obtain the required network information, such as an IP address, a default gateway, and DNS servers. 

I set the DHCP server range to 192.168.60.10-254 to leave space at the beginning of the address range in case I wanted to assign a static address to devices, such as my access point. 
<img src="/assets/PA Policies.png" />
There are various policies here because I worked on this for a few weeks before starting my write-up, but I initially started by creating the "home-to-untrusted-general-allow" policy. The "home-to-untrusted-general-allow" policy allows any traffic originating from my "home" zone to pass through to the "untrusted" zone, provided it adheres to the "application-default" service profile, which Palo Alto determines to be the standard ports for applications detected via App-ID. 
<img src="/assets/PA NAT Policies.png" />
With my "home-to-untrusted-general-allow" policy in place, traffic still did not successfully flow out to the internet. To ensure traffic could successfully reach the internet, I had to create a NAT policy that translates the private RFC 1918 addresses originating from my home zone into routable addresses on the internet. 
<img src="/assets/PA NAT translation type.png" />
For the NAT policy for traffic from my home to untrusted zones, I chose "Dynamic IP and Port" (DIPP) as the source translation type. DIPP takes the source port and IP address originating from my home zone and maps the source port to an ephemeral port that exists for the duration of the session. The source IP is translated to the WAN IP of my WAN link interface, ethernet1/1.
<img src="/assets/PA ACC.png" />
With the ACC tab filtered to my "home-to-untrusted-general-allow" security policy, we can see that traffic from my home zone is successfully reaching the internet, and sessions are being formed, confirming that my NAT rule is working as expected. 

To be continued...

## Incidents

- [Discord voice calls stuck on 'Disconnected'](Incidents/discord-voice-calls-stuck-on-disconnected.md)
