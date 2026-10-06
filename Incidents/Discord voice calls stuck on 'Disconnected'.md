## Incident
<img src="/assets/Discord-tcp-rst.png" />
My sister told me that when she tried to join a discord voice call on her computer the call would fail to connect stating "disconnected" and discord displayed an error code "2007" which according to discord (https://support.discord.com/hc/en-us/articles/30952914470807-Discord-Audio-and-Video-Error-Codes-Troubleshooting-Guide) correlates to network issues. 
## Troubleshooting Steps
I went over to my sister's computer to investigate and I recreated the issue by attempting to join a voice call, confirming that the issue was indeed present. 

While at her computer, I opened up a powershell shell and ran "ipconfig" to take note of her current ipv4 address. I went back to my own computer and opened up my PA VM web gui and went over to the monitor tab to investigate traffic originating from my sister's ip address. I noticed that my sister's ip address (192.168.60.22) was attempting to make connections to the discord application which was allowed through the firewall but the session was ending for reasons such as "tcp-rst-from-client" and "tcp-rst-from-server". 

Since I saw that the connection was allowed through the firewall to port 443, I knew it wasn't an issue with that port. My first thought was that perhaps something within my security profile group (spg) was causing the connection to Discord to drop. 

I attempted to remove the spg from the security policy my sister's traffic was matching (home-to-untrusted) but the issue remained. At that moment, I noticed that on the home-to-untrusted policy had the "application-default" profile set for the service option. I set the service option to "any" and tried to connect to a discord voice call again and it worked! 

## Root Cause
App-ID was identifying one of the Discord app dependencies, SSL, on non-standard ports which fell outside of what was within the "application-default" service profile, causing certain ports to not match my "home-to-untrusted" security policy when I had the "application-default" service profile enabled and instead fall through to my clean-up policy, "interzone-default" which blocks all traffic that it matches. 
<img src="/assets/Discord non-standard ssl.png" />
Non-standard ports identified as SSL app: 8443, 2083, 2087, 2096, etc. 
## Fix
<img src="/assets/PA Discord policy.png" />
I made a policy allowing Discord and its dependencies and set service to any to catch all of the non-standard ports. This allows app level control but not port level control which seems to be as granular as you can get without playing whack-a-mole by trying to restrict Discord and its dependencies to certain custom services. 

## Other thoughts

I hypothesis that enabling SSL decryption on my home zone would allow App-ID to better match signatures from app traffic potentially causing what is currently being identified as SSL to instead match Discord which would allow for the creation of more granular policies.
