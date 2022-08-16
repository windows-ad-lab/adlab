# Browser passwords

## Configuring

1. Download Firefox and the Chrome standalone installers.

* [https://www.mozilla.org/en-US/firefox/all/#product-desktop-release](https://www.mozilla.org/en-US/firefox/all/#product-desktop-release)
* [https://www.google.com/chrome/?standalone=1](https://www.google.com/chrome/?standalone=1)

2\. Transfer them to and install them on `DATA01`.

3\. Login with `Administrator` and the password `Welcome01!` to `DC03` and create a user with the name `bob`.

```
net user bob CredentialDumping01! /add /domain
```

![](<../../../../.gitbook/assets/image (4).png>)

4\. Login with the user `Bob` and the password `CredentialDumping01!` on `DATA01`.

### Firefox Credentials

1. Open FireFox, go to the settings by opening the menu in the right top and clicking "Settings".
2. Open the "Privacy and Security" tab and scroll down to the "Login and Passwords" section.

![](<../../../../.gitbook/assets/image (1).png>)

3\. Click on "Saved Logins" and click in the bottom on "Create New Login". Fill in the following information to save something within the Browser:

![](<../../../../.gitbook/assets/image (7).png>)

### Google Chrome

1. Open Google Chrome, go to the settings by opening the menu in the right top and clicking "Settings".
2. Click on "Autofill" and then on "Passwords"

![](<../../../../.gitbook/assets/image (3).png>)

3\. In the "Save Passwords" section click on "Add". Fill in the following information and click "Save".

![](<../../../../.gitbook/assets/image (17).png>)

## Attacking

### How it works

The saved passwords are stored in a which lets the user decrypt them without asking for a password as long as they aren't protected with a master password.

### Tools

* [DonPapi](https://github.com/login-securite/DonPAPI)

### Executing the attack

To execute the attack administrator privileges to the machine and user credentials for the users which we want to extract the credentials from are required.

1. From the Kali machine install the DonPapi tool.
2. Create a `creds.txt` file and place the credentials from `bob` in here with the `<USER>:<PASSWORD>` format. `Bob:CredentialDumping01!`

![](<../../../../.gitbook/assets/image (12).png>)

3\. Execute the following command to run DonPapi against `DATA01` using the `creds.txt` file and dumping all the credentials saved for this user on `DATA01`:

```
python3 DonPAPI.py Administrator:'Welcome01!'@10.0.0.101 -local_auth -credz creds.txt
```

![](../../../../.gitbook/assets/image.png)

We discovered the saved credentials in Chrome and Firefox.

## Defending

### Recommendations

* Don't save passwords in browsers and use a passwordmanager such as LastPass, BitWarden or KeePass.

### Detection



