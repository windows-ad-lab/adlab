# Troubleshooting

While running and building the lab we found the following issues. We noted them down here since other people might get the same issues:

## DC on private network instead of domain

If a DC is on a private network instead of the domain, the firewall will block a lot of stuff. Like DNS, and it breaks stuff. This worked for us to fix it:

[https://superit.in/dmz-rodc-is-going-in-public-network-profile-after-reboot/](https://superit.in/dmz-rodc-is-going-in-public-network-profile-after-reboot/)
