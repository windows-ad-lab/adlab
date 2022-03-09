# Attack path 1

## Attack path visualised

![1. Enumerating users with kerbrute](<../../../.gitbook/assets/1.0 enumerate users (1).png>)

![2. Password spraying with kerbrute](<../../../.gitbook/assets/2.0 Password\_spraying.png>)

![3. Escalating priviliges of our user John](<../../../.gitbook/assets/3.0 Escalate priviliges.png>)

![4. After dumping hashes we sprayed the hash against the network](<../../../.gitbook/assets/4.0 access share v2.0.png>)

## Steps for the specified attack path

1\. First we need to get a list of usernames, which are valid inside our domain. We can do this by enumerating usernames with a list of top usernames.

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/" %}
[username-enumeration](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/)
{% endcontent-ref %}

2\. After we have a list of usernames which are available in our domain, we can spray easy to guess passwords against the users that were discovered.

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md" %}
[password-spraying.md](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md)
{% endcontent-ref %}

3\. We notice some weak credentials and can now access WS01. But sadly our user has low privileges on WS01. Let's try to abuse misconfigured service's.

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md" %}
[unqouted-service-path.md](../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md)
{% endcontent-ref %}

4\. Once we have administrator privileges on WS01, we can dump the hashes. With the gathered hashes, we can spray the hash against the network and we notice IT-Support01 has access to a share on FILE01.

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md" %}
[password-spraying.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/password-spraying.md)
{% endcontent-ref %}

5\. Find PowerShell script with encrypted password

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/passwords-on-shares.md" %}
[passwords-on-shares.md](../../../vulnerabilities-misconfigurations/misc/passwords-on-shares.md)
{% endcontent-ref %}

6\. Abuse the permission this user has to reset the password of \<USER>.



7\. \<USER> has constrained delegation to FILE01. Abuse this to get access to the server.

{% content-ref url="../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md" %}
[constrained-delegation.md](../../../vulnerabilities-misconfigurations/active-directory-attacks/delegation-attacks/constrained-delegation.md)
{% endcontent-ref %}

8\. `FILE01` had Domain Admin logged on and is able to access `DC01`.

