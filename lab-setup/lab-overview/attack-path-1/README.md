# Attack path 1

## Attack path visualized

![](../../../.gitbook/assets/attack\_path1.drawio.png)

## Tasks for the specified attack path

* [ ] Enumerate valid usernames.
* [ ] Spray password against the valid usernames discovered.
* [ ] Try to get access to the workstation machines.
* [ ] Escalate privileges by abusing a mis-configured service.
* [ ] Dump credentials and enumerate shares.
* [ ] Retrieve credentials from a share.
* [ ] Enumerate ACL's for this user and abuse them.
* [ ] Abuse constrained delegation.
* [ ] Abuse Unconstrained delegation with the printerbug.
* [ ] Escalate privileges to the parent domain.
* [ ] Discover Foreign Security Principals and access a share across trusts.
* [ ] Retrieve credentials from the share.
* [ ] Try to get access to the fileserver.
* [ ] Abuse ACL and execute DC Sync

## Pages

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/" %}
[username-enumeration](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md" %}
[password-spraying.md](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md" %}
[unqouted-service-path.md](../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/passwords-on-shares.md" %}
[passwords-on-shares.md](../../../vulnerabilities-misconfigurations/misc/passwords-on-shares.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/acl-abuses/page-1.md" %}
[page-1.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/acl-abuses/page-1.md)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md" %}
[constrained-delegation.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md)
{% endcontent-ref %}

