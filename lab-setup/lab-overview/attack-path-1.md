# Attack path 1

## Attack path visualised

//ToDo: image

## Steps for the specified attack path

1\. First we need to get a list of usernames, which are valid inside our domain. We can do this with enumerating usernames. We will use the tool kerbrute.

{% content-ref url="../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/" %}
[username-enumeration](../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/)
{% endcontent-ref %}

2\. After we have a list of usernames which are available in our domain, we can spray easy to guess passwords against it.

{% content-ref url="../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md" %}
[password-spraying.md](../../vulnerabilities-misconfigurations/initial-access-attacks/username-enumeration/password-spraying.md)
{% endcontent-ref %}

\


3\. Ws01 privesc with misconfigured service

4\. Dump hashes - IT user logged in who can access share and IT folder on File01

5\. Power Shell script with encrypted password (or another configuratie file or anything)

6\. The user has resetpassword on multiple users including a \<USER>

7\. \<USER> has constrained delegation to file01

8\. File01 had Domain Admin logged on and is able to access dc01
