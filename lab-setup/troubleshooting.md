# Troubleshooting

While running and building the lab we found the following issues. We noted them down here since other people might get the same issues:

## DC on private network instead of domain

If a DC is on a private network instead of the domain, the firewall will block a lot of stuff. Like DNS, and it breaks stuff. This worked for us to fix it:

{% embed url="https://superit.in/dmz-rodc-is-going-in-public-network-profile-after-reboot/" %}

## PSRemoting not available on WS01

* Run the Enable-Psremoting cmdlet again.
* Add a extra firewall rule allowing all traffic to port tcp 5985.

![](<../.gitbook/assets/image (72) (1) (1).png>)
