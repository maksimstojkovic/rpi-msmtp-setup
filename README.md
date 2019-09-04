# MSMTP Mail Client Docs
This document will outline the steps taken to setup msmtp on a Raspberry Pi Zero W for use with services such as Unattended Upgrades and Logwatch.

## Mail Setup
Install required packages:
```
sudo apt-get install msmtp msmtp-mta bsd-mailx update-notifier-common
```

Create an msmtp config file with the following settings:
```
sudo nano /etc/msmtprc
```

```
# Set default values for all following accounts.
# Set default values for all following accounts.
defaults
auth           on
tls            on
tls_starttls   on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /tmp/msmtp.log

# Gmail
account        gmail
host           smtp.gmail.com
port           587
auth           login
from           user@gmail.com
user           user@gmail.com
password       pass

# Set a default account
account default : gmail
```

Test whether the root user can send an email using the mailx command:
```
echo "This is the email body" > /tmp/body.txt && sudo mailx -s "This is the subject" <YOUR_EMAIL>@gmail.com < /tmp/body.txt; rm /tmp/body.txt
```

## Unattended Upgrades
Edit unattended-upgrade settings to send emails following updates:
```
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

The following two lines should be the only uncommented lines in the `Origins-Pattern` section:
```
"origin=Raspbian,codename=${distro_codename},label=Raspbian";
"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
```

Edit the mail settings as follows:
```
Unattended-Upgrade::Mail "<YOUR_EMAIL>@gmail.com";
Unattended-Upgrade::MailOnlyOnError "false";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
Unattended-Upgrade::Automatic-Reboot-Time "05:00";
```
**Note:** The package `update-notifier-common` must be installed for automatic reboots to take place.

## Logwatch
Ensure that the `logwatch.conf` file has been copied from the isntall directory `/usr/share/logwatch/default.conf` to `/etc/logwatch/conf`:
```
sudo cp /usr/share/logwatch/default.conf/logwatch.conf /etc/logwatch/conf/
```

Edit the settings in the `logwatch.conf` file as follows:
```
sudo nano /etc/logwatch/conf/logwatch.conf
```

```
Format = html
MailTo = <YOUR_EMAIL>@gmail.com
Detail = High
Detail = Med
mailer = "/usr/bin/msmtp -t"
```
**Note:** The most important line is the `mailer` line. No emails will be sent out if the line is not modified.

When testing logwatch with email output, use the following command:
```
sudo logwatch --output mail
```

## Useful Links
[msmtp - ArchWiki](https://wiki.archlinux.org/index.php/Msmtp)
[Raspberry Pi Stack Exchange - Unattended Upgrades on Raspbian Stretch](https://raspberrypi.stackexchange.com/questions/72022/configuring-unattended-upgrades-on-raspbian-stretch)
[Logwatch - Community Help Wiki](https://help.ubuntu.com/community/Logwatch)
