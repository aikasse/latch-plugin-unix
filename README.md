> ###WARNING###
>Please note that the explotation of the recent Shellshock vulnerability [https://shellshocker.net/](https://shellshocker.net/) allows potential attackers to bypass the Latch plugin in unpatched systems. Please update as soon as possible if you haven't done so yet.

#LATCH INSTALATION GUIDE FOR UNIX


##PREREQUISITES
* Update repositories.
For Ubuntu/Debian,
```
sudo apt-get update
```
For RedHat/CentOS/Fedora,
```
sudo yum update
```

* C compilator gcc and make.
For Ubuntu/Debian,
```
sudo apt-get install gcc make
```
For RedHat/CentOS/Fedora,
```
sudo yum install gcc make
```

* Libraries: (libpam-dev, libcurl-dev, libssl-dev).
For Ubuntu/Debian,
```
sudo apt-get install libpam0g-dev libcurl4-openssl-dev libssl-dev
```
For RedHat/Centos/Fedora,
```
sudo yum install pam-devel libcurl-devel openssl-devel
```
For FreeBSD,
Because of some problems with libcurl library, you must disable threaded_resolver[] option before make install:  
```
cd /usr/ports/ftp/curl/
make config
```
And then, install curl using ports:  
```
make install clean
```



* To get the **"Application ID"** and **"Secret"**, (fundamental values for integrating Latch in any application), it’s necessary to register a developer account in [Latch's website](https://latch.elevenpaths.com). On the upper right side, click on **"Developer area"**.


##DOWNLOADING THE UNIX PLUGIN
* When the account is activated, the user will be able to create applications with Latch and access to developer documentation, including existing SDKs and plugins. The user has to access again to [Developer area](https://latch.elevenpaths.com/www/developerArea), and browse his applications from **"My applications"** section in the side menu.

* When creating an application, two fundamental fields are shown: **"Application ID"** and **"Secret"**, keep these for later use. There are some additional parameters to be chosen, as the application icon (that will be shown in Latch) and whether the application will support OTP  (One Time Password) or not. The user can add as operations as services wants to protect with latch. When editing latch.conf configuration file, user must use these **"Operation ID's"**.

* From the side menu in developers area, the user can access the **"Documentation & SDKs"** section. Inside it, there is a **"SDKs and Plugins"** menu. Links to different SDKs in different programming languages and plugins developed so far, are shown.


##INSTALLING THE PLUGIN IN UNIX
* Cd to the top-level directory of the plugin, and use the **"./configure prefix=/usr sysconfdir=/etc && make && sudo make install"** command to install it.
```
./configure prefix=/usr sysconfdir=/etc && make && sudo make install
```

If you are installing on OpenBSD/FreeBSD, add CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" directives to "configure", since gcc will not find dependencies otherwise.
```
./configure CFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib" prefix=/usr sysconfdir=/etc && make && make install
```

* Edit /etc/latch/latch.conf file and add your **"Application ID"** and **"Secret"**. Add as operations as services will be protected with latch.

* The action option within latch.conf file (that may be open/close) specifies the action to be performed in case there is no connection to Latch servers.

* Move pam_latch.so (located in $distdir/lib) into the PAM directory (where PAM modules are stored).
```
sudo mv /usr/lib/pam_latch.so $PAM_DIR
```

Depending on the system, PAM directory is located in a different place:

OSX:
```
PAM_DIR=/usr/lib/pam
```
Ubuntu, Debian:
```
PAM_DIR=/lib*/*/security
```
CentOS, Fedora, RedHat:
```
PAM_DIR=/lib*/security/
```
FreeBSD (default installation directory):
```
PAM_DIR=/urs/lib/
```

* There are some PAM configuration examples how to protect some applications (such as sudo, sshd, su, login, etc.) in examples/ directory. Usually, your PAM module is setup by adding a line to the appropriate file in /etc/pam.d/:
```
auth	   required	pam_latch.so config=/etc/latch/latch.conf accounts=/etc/latch/latch.accounts operation=operation otp=yes
```
If you look at 'operation' parameter, it is defined the necessary operation which must be defined in latch.conf with its operation_id.



##INSTALLING THE PLUGIN IN SSH
* Edit sshd_config (usually in /etc or /etc/ssh) to add the following lines:
```
UsePAM yes
ChallengeResponseAuthentication yes
PasswordAuthentication no
```

* In order to protect authentication for SSH pubkeys, use the command option in users’ ~/.ssh/authorized_keys:
```
command="latch-ssh-cmd -o sshd-keys" ssh-rsa AAA...HP5 someone@host
```

Note: OTP not implemented for latch-ssh-cmd.

* Restart ssh service.

For Ubuntu/Debian,
```
sudo service ssh restart
```
For RedHat/CentOS/Fedora/FreeBSD,
```
sudo service sshd restart
```


###SELinux (Fedora/CentOS) SETUP
* In some systems, like Fedora 20 and CentOS 6.7, the program **“SELinux“** at times defines a security policy that prevents communication from being opened between the SSH server and the Latch server. To solve this problem, you must add a SELinux module to the policy. To do so you must enter the **“modules/SSH/SELinux“** folder of the packet for the downloaded plugin and execute the command **“semodule -i latch_ssh.pp“**. Then you must enable the variable that was created through the command **“setsebool -P ssh_can_network 1“**.


##UNINSTALLING THE PLUGIN IN UNIX
* Re-configure PAM files, sshd_config.

* Open a terminal. Move to the top-level directory of the plugin. Run **"sudo make uninstall"**.
```
./configure prefix=/usr sysconfdir=/etc && make && sudo make uninstall
```

##USE OF LATCH PLUGIN FOR THE USERS
**Latch does not affect in any case or in any way the usual operations with an account. It just allows or denies actions over it, acting as an independent extra layer of security that, once removed or without effect, will have no effect over the accounts, which will remain with their original state.**

The user needs the Latch application installed on the phone, and follow these steps:

* **Step 1:** From the Latch app on the phone, the user has to generate the token, pressing on **“generate pairing code to add service"** at the bottom of the application.

* **Step 2:** Pairing with command line: Just open a console and execute **“latch -p TOKEN_GENERATED_BY_LATCH”**.

* **Step 3:** Now the user may lock and unlock the account, preventing any unauthorized access.


* For unpairing from command line, use "latch -u" command.
