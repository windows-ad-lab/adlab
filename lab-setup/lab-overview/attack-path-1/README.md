# Attack path 1

## Attack path visualised

![1. Enumerating users with kerbrute](<../../../.gitbook/assets/1.0 enumerate users (1).png>)

![2. Password spraying with kerbrute](<../../../.gitbook/assets/2.0 Password\_spraying.png>)

## Steps for the specified attack path

1\. First we need to get a list of usernames, which are valid inside our domain. We can do this by enumerating usernames with a list of top usernames.

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/" %}
[username-enumeration](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/)
{% endcontent-ref %}

2\. After we have a list of usernames which are available in our domain, we can spray easy to guess passwords against the users that were discovered.

{% content-ref url="../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md" %}
[password-spraying.md](../../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md)
{% endcontent-ref %}

3\. Ws01 privesc with misconfigured service

{% content-ref url="../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md" %}
[unqouted-service-path.md](../../../vulnerabilities-misconfigurations/misc/misconfigured-service/unqouted-service-path.md)
{% endcontent-ref %}



4\. Dump hashes - IT user logged in who can access share and IT folder on File01

5\. Power Shell script with encrypted password (or another configuratie file or anything)

6\. The user has resetpassword on multiple users including a \<USER>

7\. \<USER> has constrained delegation to file01

8\. File01 had Domain Admin logged on and is able to access dc01
