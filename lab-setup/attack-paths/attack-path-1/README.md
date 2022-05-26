# Attack path 1

## Attack path visualized

![](../../../.gitbook/assets/attack\_path1.drawio.png)

## Tasks for the specified attack path

1. Enumerate valid usernames.
2. Spray password against the valid usernames discovered.
3. Try to get access to the workstation machines.
4. Escalate privileges by abusing a mis-configured service.
5. Dump credentials and enumerate shares.
6. Retrieve credentials from a share.
7. Enumerate ACL's for this user and abuse them.
8. Abuse constrained delegation.
9. Abuse Unconstrained delegation with the printerbug.
10. Escalate privileges to the parent domain.
11. Discover Foreign Security Principals and access a share across trusts.
12. Retrieve credentials from the share.
13. Try to get access to the fileserver.
14. Abuse ACL and execute DC Sync

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

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md" %}
[constrained-delegation.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md)
{% endcontent-ref %}

// Need to add extra pages.
