# DACL-Abuses

Active Directory Discretionary Access Control Lists (DACLs) and Acccess Control Entries (ACEs) that make up DACLs. The best information out there on how to abuse these edges will be on the [BloodHound wiki](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#). Here s a short list of interesting ACL abuses:

* **GenericAll** - full rights to the object. Modify group membership, modify user attributes (Set SPN to kerberoast) or reset a user their password. Also possible to read LAPS passwords of a computerobject.
* **GenericWrite** - update object's attributes (i.e logon script, serviceprincipalname, group membership.
* **WriteOwner** - change object owner to attacker controlled user take over the object. When owner you can write generic all to the object.
* **WriteDACL** - ability to modify the DACL on the target object, you can grant yourself almost any privilege against the object you wish.
* **AllExtendedRights** - ability to add user to a group or reset password.
* **ForceChangePassword** - ability to change a user their password.
