# Resource Based Constrained Delegation

In resource based Kerberos delegation, computers (resources) specify who they trust and who can delegate authentications to them. The attacks abuses this by writing to the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute that a user or computer that we control can impersonate and authenticate as any domain user to the computer.

So lets say that `WEB01` trusts `FAKE01` due to the modified `AllowedToActOnBehalfOfOtherIdentity` atribute. `FAKE01` can impersonate any users that is trusted for delegation and can authenticate as Domain Admin to `WEB01`.

The attack can (as far as I know) be performed in three ways:

* By abusing WRITE privileges on a computer object, it is possible to write to this attribute.
* By triggering the target workstation to authenticate over webdav to the attacker machine and relay the hash to LDAP. Writing to this attribute. Forcing to authenticate can be done:&#x20;
  * From a low-priv shell by abusing Change-LockScreen.
  * If the webclient is active with petitpotam or printerbug.py.

Here are the articles in this section:
