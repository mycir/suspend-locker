# suspend-locker

A suspend alternative to KDE kscreenlocker, which may be problematic for some laptops:

* Suspend/resume hangs or freezes.

* Ryzen 5/Radeon graphics - some versions of AMD driver show brief screen corruption or break compositor on resume.

Applied successfully to several different HP laptops under Kubuntu 18.04, 20.04 and 22.04.

## Dependencies

i3lock, xinput and xset

If not already installed:

```
sudo apt install i3lock xinput x11-xserver-utils
```

## Installation
Copy required files:

suspend-locker > /usr/local/bin  
suspend-locker.conf > /etc &| $HOME/.config  
fooprints.png|<my_image.png*> > /usr/share/backgrounds &| $HOME/.local/share/wallpapers  
suspend-locker.service > /etc/systemd/system
<br></br>
Change files owner and mode:
```
chown root:root /usr/local/bin/suspend-locker  
chmod 744 /usr/local/bin/suspend-locker  
chown root:root /etc/suspend-locker.conf  
chmod 644 /etc/suspend-locker.conf
chown root:root /etc/systemd/system/suspend-locker.service
chmod 644 /etc/systemd/system/suspend-locker.service
```  
Disable System Settings > Workspace Behaviour > Screen Locking, After waking from sleep.
<br></br>
Finally, execute:

```
sudo systemctl enable suspend-locker.service
sudo systemctl daemon-reload
```  
*Amend Wallpaper=<my_image.png> in suspend-locker.conf

## Notes

NB suspend-locker is for personal use and is not intended as a 'maximum security' solution - see [Martin's Blog - A sandbox for the screen locker](https://blog.martin-graesslin.com/blog/tag/kscreenlocker/)

kscreenlocker does actually 'behave' in the context of suspend-locker. If preferred, set Locker=kde in suspend-locker.conf

/etc/systemd/logind.conf and /etc/systemd/sleep.conf|/etc/acpi/events/sleep.conf may require some tweaking to handle the lid switch. For example, on an HP 17-ca1027na running Kubuntu 22.04, the commented out defaults work fine. Whereas a veteran HP550 running Kubuntu 18.04 required the following changes:
<br></br>
<ins>/etc/systemd/logind.conf</ins>  
...  
HandleLidSwitch=suspend  
...
<br></br>
<ins>/etc/acpi/events/sleep.conf</ins>  
...  
HandleLidSwitch=suspend  
HandleLidSwitchDocked=suspend  
LidSwitchIgnoreInhibited=yes  
...
<br></br>
<ins>/etc/acpi/events/lid</ins>  
event=button/lid LID close
action=/etc/acpi/lid.sh
<br></br>
<ins>/etc/acpi/lid.sh</ins>  
#!/bin/bash
xset dpms force off
loginctl lock-sessions  
dbus-monitor --session "type='signal',interface='org.freedesktop.ScreenSaver'" |  
&nbsp;&nbsp;&nbsp;while read x; do  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[[ $x =~ "boolean true" ]] && pkill -f "dbus-monitor"  
&nbsp;&nbsp;&nbsp;done  
systemctl suspend
