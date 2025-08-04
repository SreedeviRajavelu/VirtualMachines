## Setting up Ubuntu VM on VirtualBox on MacBook M4 :
https://www.youtube.com/watch?v=LjL_N0OZxvY

## Make Ubuntu full screen in VirtualBox:
https://youtu.be/Fw8ppXeJ_GY?si=qu0ehtCFotNpaLcB
- this tutorial also shows how to install guest additions

## How to create shared folder between host and Ubuntu VM on VirtualBox:
https://youtu.be/j8Ne96h8UDg?si=aBwXLuINWmvPNp1l


Fix slow booting for ubuntu vm based on output of these commands to find what was slowing down the booting time


     systemd-analyze
     systemd-analyze blame

sreedevi@sreedevi:~/Desktop$ systemd-analyze blame
2min 258ms systemd-networkd-wait-online.service
    1.782s fwupd.service
    1.622s snapd.seeded.service
    1.578s snapd.service
    1.124s snap.docker.nvidia-container-toolkit.service
     710ms NetworkManager.service
     704ms vboxadd.service
     335ms dev-mapper-ubuntu\x2d\x2dvg\x2dubuntu\x2d\x2dlv.device
     290ms gnome-remote-desktop.service
     241ms NetworkManager-wait-online.service
     231ms power-profiles-daemon.service
     222ms accounts-daemon.service
     217ms polkit.service
     214ms snapd.apparmor.service
     213ms udisks2.service
     187ms rsyslog.service
     179ms apparmor.service
     167ms apport.service
     163ms grub-common.service
     142ms avahi-daemon.service
     132ms ModemManager.service
     130ms dev-loop1.device
     128ms systemd-udev-trigger.service
lines 1-23

Top delay:
2min 258ms systemd-networkd-wait-online.service

This service alone adds over 2 minutes to your boot. That’s practically your entire userspace delay.
🔍 What is systemd-networkd-wait-online.service?
It waits for the network to be fully up (like getting a DHCP lease or full connection).
Used mostly on servers to ensure critical services only start after network is available.
In most VMs and desktops, it's not necessary and can be safely disabled.
✅ Fix: Disable the Wait-Online Service
Run:
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
💡 mask ensures it cannot be accidentally started again.


If you also see or use NetworkManager, you can disable its wait service too:
sudo systemctl disable NetworkManager-wait-online.service
sudo systemctl mask NetworkManager-wait-online.service
🔁 Then reboot your VM:
sudo reboot
🧪 After reboot:
Check again:
systemd-analyze
You should see your boot time drop from 2+ minutes → ~15 seconds or less.

🔁 To Undo the Effects of Disabling and Masking the Wait-Online Service
If you previously ran:

- sudo systemctl disable systemd-networkd-wait-online.service

- sudo systemctl mask systemd-networkd-wait-online.service
  
You can restore the service like this:

✅ Step 1: Unmask the service

- sudo systemctl unmask systemd-networkd-wait-online.service

✅ Step 2: Re-enable the service at boot

- sudo systemctl enable systemd-networkd-wait-online.service
✅ Step 3 (Optional): Start it immediately (not needed for reboot use)

sudo systemctl start systemd-networkd-wait-online.service

🔁 Same steps apply if you disabled NetworkManager-wait-online.service:
Just replace the service name in each command.
Example:

sudo systemctl unmask NetworkManager-wait-online.service

sudo systemctl enable NetworkManager-wait-online.service

🧪 Confirm it's active again:

systemctl status systemd-networkd-wait-online.service
or
systemctl is-enabled systemd-networkd-wait-online.service

You should see:

`enabled`

✅ Summary: Rollback Commands

# Unmask and re-enable systemd-networkd-wait-online

sudo systemctl unmask systemd-networkd-wait-online.service

sudo systemctl enable systemd-networkd-wait-online.service

# Optional: Also for NetworkManager-wait-online (if previously masked)

sudo systemctl unmask NetworkManager-wait-online.service

sudo systemctl enable NetworkManager-wait-online.service
