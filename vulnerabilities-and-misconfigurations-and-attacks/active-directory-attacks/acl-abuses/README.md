# DACL-Abuses

Active Directory Discretionary Access Control Lists (DACLs) and Acccess Control Entries (ACEs) that make up DACLs. The best information out there on how to abuse these edges will be on the [BloodHound wiki](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html). Here s a short list of interesting ACL abuses:

* **GenericAll** - full rights to the object. Modify group membership, modify user attributes (Set SPN to kerberoast) or reset a user their password. Also possible to read LAPS passwords of a computerobject.
* **GenericWrite** - update object's attributes (i.e logon script, serviceprincipalname, group membership.
* **WriteOwner** - change object owner to attacker controlled user take over the object. When owner you can write generic all to the object.
* **WriteDACL** - ability to modify the DACL on the target object, you can grant yourself almost any privilege against the object you wish.
* **AllExtendedRights** - ability to add user to a group or reset password.
* **ForceChangePassword** - ability to change a user their password.

## BloodHound

The easiest way to check for ACL abuses is using BloodHound since it will visualize the ACL abuses and it is easy to to check if any new owned objects has ACL's. The easiest way to check it is searching for the user at the top and clicking on the user, then scroll down all the "Node Info" and look for "Outbound control rights". This is the section thats hows if the user has any ACL's:

![](<../../../.gitbook/assets/image (40) (1) (1).png>)

The object has one First Degree Object Control, meaning the user has one direct ACL to another object. If the user is member of a group and that group had a ACL to another object it would show under "Group Delegated Object Control". Once you click on the number it will show the ACL's:

![](<../../../.gitbook/assets/image (20) (1) (1) (1).png>)

I will always recommend doing this for EVERY user and computer or other objects you got access to and own(Know the password from or NTLM hash or tickets etc). And also right click on the objects and mark them as "Owned"

![](<../../../.gitbook/assets/image (44) (1).png>)

{% embed url="https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html" %}
