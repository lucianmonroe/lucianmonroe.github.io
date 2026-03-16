TUTORIAL BY LUCIAN MONROE
March 13, 2026 (http://lucianmonroe.github.io)
Section 1: How to Download the ISO Image
Go to the official Void Linux website.
https://voidlinux.org/download/
Click on the Download section.
You'll see a list of ISO images. Find the one called glibc version with XFCE.
Click the download link. Wait for it to finish.
Section 2: How to Make the USB Installer
Plug in your USB drive.
Open Terminal.
Run this command to list all your disks:
diskutil list
Find your USB drive in the list. It'll look something like /dev/disk2 — remember that number.
⚠️ Always double-check your disk number. Picking the wrong one will wipe the wrong drive.
Unmount the USB drive:
diskutil unmountDisk /dev/disk2
Now flash the ISO onto the USB:
sudo dd if=/Users/barn/Downloads/void-live-x86_64-20250202-xfce.iso of=/dev/rdisk2 bs=1m
Note: The path and filename above is just an example. Use your actual file path and filename.
⚠️ Replace disk2 and rdisk2 with your actual disk number from the list.
Wait for it to finish. It'll look frozen, that's normal. Don't unplug anything.
If you're impatient with no progress bar in sight, press Ctrl + T to check how many bytes have been copied so far.
Once it's done, macOS may pop up saying the disk is not recognized. Just click Eject. Or eject it manually yourself.
Section 3: Running the Installer
Open Terminal and run:
sudo void-installer
Follow through each section:
Keyboard/Locale Set these to your preference. Usually us and en_US.UTF-8.
Network Select WiFi. Connect to your network.
Source Select Local. This installs packages directly from the USB.
Hostname Type a name for your computer. It can be anything you want.
Partitions This opens cfdisk.
●	Highlight your partitioned drive (example: 60GB — sda4). Select Delete.
●	Select New. Make sure it's set to Linux filesystem.
●	Select Write, type yes, then select Quit.
Filesystems This is the most important part. Do this carefully.
●	Find sda1 (200MB): Set mount point to /boot/efi. Set filesystem to vfat. When asked "Do you want to create a new filesystem on this partition?" select No. This means you are NOT formatting it.
●	Find sda4 (60GB): Set mount point to /. Set filesystem to ext4. When asked "Do you want to create a new filesystem on this partition?" select Yes. This means you ARE formatting it.
Section 4: Update, Install, and Configure Keys
Update and Install
Run these one by one:
sudo xbps-install -u xbps
sudo xbps-install -Su
sudo xbps-install -S nano keyd
keyd is a key remapping tool. It lets you reassign what your keys do at the system level.
Configure Your Keys
Create the config folder (only needed once):
sudo mkdir -p /etc/keyd
Open the config file:
sudo nano /etc/keyd/default.conf
Paste this into the file:
[ids]
*

[main]
7 = y
8 = u
9 = i
0 = o
- = p
p = -
previoussong = 7
playpause = 8
nextsong = 9
mute = 0
Press Ctrl+O, Enter, then Ctrl+X to save and exit.
Enable the Service
On Void, installing a package doesn't start the service automatically. You have to link it manually:
sudo ln -s /etc/sv/keyd /var/service/
Apply your changes immediately:
sudo sv restart keyd

Section 5: Install GNOME and Display Manager
Run this to install GNOME, essential apps, GDM, and the browser connector for extensions:
sudo xbps-install -S gnome gdm gnome-browser-connector
Install Required Services & Tools
Run this to install dbus, elogind, and NetworkManager:
sudo xbps-install -y dbus elogind
Now open the GRUB config file:
sudo nano /etc/default/grub
Find the line that says GRUB_CMDLINE_LINUX_DEFAULT and change the value to loglevel=2.
[ss2.png]
loglevel=2 is needed because on Void with GNOME, using the default loglevel=4 causes errors on boot.
Press Ctrl+O, Enter, then Ctrl+X to save and exit.
Apply the changes:
sudo update-grub
Enable Services
Link the necessary services to start at boot:
sudo ln -s /etc/sv/dbus /var/service/
sudo ln -s /etc/sv/gdm /var/service/
sudo ln -s /etc/sv/NetworkManager /var/service/
sudo ln -s /etc/sv/elogind /var/service/
Remove lightdm. We don't need XFCE anymore, because we're switching to GDM. This removes lightdm from starting at boot:
sudo rm /var/service/lightdm
Reboot
sudo reboot
When your system boots back up, select GNOME Wayland from the login screen.
Section 6: Theming
Cursor
Use the default Adwaita cursor theme.
Run this to set the cursor size:
gsettings set org.gnome.desktop.interface cursor-size 24
Icons
Download the Colloid Light icons.
https://www.gnome-look.org/p/1661983
Create the icons folder:
.icons
Move the Colloid icon folder into .icons.
Open GNOME Tweaks. Set the icons under Appearance.
GTK Theme
Download the Marble Shell (blue-light).
https://www.gnome-look.org/p/1977647/
Create the themes folder:
.themes
Move the Marble theme folder into .themes.
Install the User Themes extension. Then open GNOME Tweaks and set the theme under Appearance.
Wallpaper 
Download the wallpaper from: https://4kwallpapers.com/nature/sand-dunes-desert-landscape-evening-windows-10x-microsoft-3287.html Right-click the image and save it. Go to Settings > Background and set it as your wallpaper.
Section 7: GNOME Extensions
Open Firefox and download these extensions:
●	Blur my Shell: Open the Blur my Shell extension settings. Go to the Pipeline section. Create a new pipeline. Set the effect to Color. Set transparency to zero for both Panel and Overview.
●	Dash to Panel 
●	Hide Top Bar
●	Just Perfection
●	Tiling Assistant
●	Clipboard Indicator
Section 8: Clean Up, Flatpak, and App Installs
Remove XFCE. You don't need it anymore:
sudo xbps-remove -R xfce4 xfce4-plugins
Then run:
Install Flatpak:
sudo xbps-install -S flatpak
Add Flathub:
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
How to Install Flatpak Apps Search for an app via GNOME Software or Google to find its application ID. Or search in terminal: flatpak search [name] Copy the application ID and install it: flatpak install flathub [application ID] To uninstall: flatpak uninstall [application ID]
If you would like to download GNOME Software run:
sudo xbps-install -S gnome-software gnome-software-plugin-flatpak
Install via xbps
1.	Krita
2.	Flameshot
3.	MyPaint
4.	GNOME Todo
5.	Xournal++
6.	VS Code
7.	git
8.	noto-fonts-emoji
sudo xbps-install -S krita flameshot mypaint gnome-todo xournalpp git noto-fonts-emoji
Optional: Inkscape, Blender, OpenToonz
sudo xbps-install -S inkscape blender opentoonz
Install via Flatpak
1.	OnlyOffice
2.	Discord
3.	Solanum
flatpak install flathub com.vscodium.codium flatpak install flathub org.onlyoffice.desktopeditors flatpak install flathub com.discordapp.Discord flatpak install flathub org.gnome.Solanum
Manual Install
1.	XP Pen Driver (see Section 11)
Section 9: Optional Extras
Bluetooth
Install bluez and enable the service:
sudo xbps-install -S bluez
sudo ln -s /etc/sv/bluetoothd /var/service/
See Key Input
To see what keys you're pressing on your device, install libinput:
sudo xbps-install -S libinput
Run this to start watching key events:
sudo libinput debug-events
Press Ctrl+C to stop.
Section 10: Screenshots and Shortcuts
Screenshot Shortcuts 
Go to Settings > Keyboard > View and Customize Shortcuts. 
Go to Screenshots. Set: 
→ Full screen screenshot: Ctrl + Shift + 3 
→ Selection screenshot: Ctrl + Shift + 4
Flameshot for Annotating 
Make sure Flameshot is installed (it should be from Section 8). 
Go to Settings > Keyboard > View and Customize Shortcuts. 
Scroll down to Custom Shortcuts.
 Add a new one: 
→ Name: Flameshot 
→ Command: flameshot gui 
→ Shortcut: Ctrl + Shift + 5
Section 11: XP Pen G640S Driver
Download the driver tar.gz file from the XP Pen website. Extract it manually using GNOME Files. 
Copy the path of the extracted folder. Open Terminal and navigate to that folder: cd /home/void/Downloads/XPPenLinux4.0.13-251226 
Replace the path with your actual extracted folder path. 
Run the installer: sudo ./install.sh
Section 12: Download the .docx or .pdf file of this documentation
gnomevoid.docx 
gnomevoid.png

