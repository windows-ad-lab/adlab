# Resource Based Constrained Delegation

In resource based Kerberos delegation, computers (resources) specify who they trust and who can delegate authentications to them. The attacks abuses this by writing to the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute that a user or computer that we control can impersonate and authenticate as any domain user to the computer.

So lets say that `WEB01` trusts `FAKE01` due to the modified `AllowedToActOnBehalfOfOtherIdentity.` `FAKE01` can impersonate any users that is trusted for delegation and can authenticate as Domain Admin to `WEB01`.



Here are the articles in this section:
