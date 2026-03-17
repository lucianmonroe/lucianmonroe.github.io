# lucianmonroe.github.io
# Install the wallpapers and icon-themes.zip for quick acsess to the downloads
![My Screenshot](ss.png)
# To make gnome software work use this : 
It looks like you're missing a few essential "glue" packages that GNOME Software needs to actually talk to the internet and parse the app data. Since `killall` and `appstream-util` aren't found, your installation is a bit "too" minimal right now.

Let's get the necessary tools installed and fix the background process.

### 1. Install Missing Utilities

You need `psmisc` (for the `killall` command) and `appstream-glib` (for the metadata tool). Run this:

```bash
sudo xbps-install -S psmisc appstream-glib

```

### 2. The Correct "Nuclear" Reset

Since `gnome-software --verbose` didn't show anything, the process might be hung in the background. Do this exactly:

1. **Kill the process** (now that you have `killall`):
```bash
killall gnome-software

```


2. **Clear the app data AND the internal database**:
```bash
rm -rf ~/.cache/gnome-software
rm -rf ~/.local/share/gnome-software

```


3. **Force the AppStream refresh**:
```bash
sudo appstream-util refresh-cache --force

```



### 3. Check for the Flatpak Plugin

You mentioned it usually uses Flatpak. On Void, the plugin is a separate package. If this is missing, GNOME Software will be a "ghost town." Ensure it's there:

```bash
sudo xbps-install -S gnome-software-plugin-flatpak

```

### 4. Test the Search Connection

Now, run GNOME Software again. It will take a minute to "re-index" Flathub:

```bash
gnome-software --verbose

```

**Keep an eye on the terminal output.** Look for any lines that say:

* `flatpak: failed to load...`
* `appstream: no data found...`

If you see those, it usually means your Flathub "remote" is added but hasn't actually downloaded the metadata yet. You can force that with:

```bash
flatpak update --appstream

```

It’s a great feeling when those clicks finally start firing! You’ve essentially built a custom sound engine on a minimalist "rolling" system—not a small feat.

Here is the complete summary of the steps we took to get **Keyboard Sounds** running on **Void Linux (GNOME 48.5/49)**.

---

## 🛠️ The Ultimate Keyboard Sound

### Phase 1: Install Build Tools & Libraries
Because Void Linux is minimal, you first need the "blueprints" to compile the sound engine from source code.
```bash
sudo xbps-install -S base-devel python3-devel kernel-libc-headers \
SDL2-devel SDL2_mixer-devel SDL2_image-devel SDL2_ttf-devel \
freetype-devel libpng-devel portmidi-devel python3-tkinter
```

### Phase 2: Install the Python Package Manager
You need `pipx` to install the tool in an isolated "bubble" so it doesn't break your system.
```bash
sudo xbps-install -S python3-pipx
```

### Phase 3: Build the Sound Engine
This part takes the longest because it compiles the `pygame` and `evdev` libraries specifically for your hardware.
```bash
pipx install keyboardsounds --pip-args="pygame-ce"
```

### Phase 4: Configure the "Path"
You must tell your terminal where the new `kbs` command lives.
1. Run this: `pipx ensurepath`
2. Then refresh your shell: `source ~/.bashrc`
3. Verify it works: `kbs --version` (It should return a version number like **6.4.3**).

### Phase 5: Fix System Permissions
Wayland and GNOME require your user to have permission to "hear" the keyboard across different apps.
1. Add your user to the input group: `sudo gpasswd -a $USER input`
2. **CRITICAL:** You must **Log Out and Log Back In** for this to apply.

### Phase 6: The GNOME Extension
1. Install **Extension Manager** from the software store or via terminal: `sudo xbps-install -S extension-manager`.
2. Disable version checking (since you are on GNOME 48.5 and the extension is for 49):
   ```bash
   gsettings set org.gnome.shell disable-extension-version-validation true
   ```
3. Open **Extension Manager**, search for **Keyboard Sounds**, and turn it **ON**.
---

## 🛠️ The Ultimate Mouse Sound

### Phase 1: The Foundation (Udev Rules)
This step is critical because it gives your user account permission to "hear" the hardware without needing a password. This is what allows the sounds to start automatically on boot.

1.  **Open the rules file:**
    `sudo nano /etc/udev/rules.d/99-mechanical.rules`
2.  **Paste these exact lines:**
    ```text
    KERNEL=="event8", MODE="0666"
    KERNEL=="event23", MODE="0666"
    ```
3.  **Apply the rules immediately:**
    ```bash
    sudo udevadm control --reload-rules && sudo udevadm trigger
    ```

---

### Phase 2: The Brain (`~/mouse_driver.py`)
This Python script handles the "Smart Tap" logic (the 150ms window) and the volume balance between light taps and heavy force clicks.

1.  **Open the file:** `nano ~/mouse_driver.py`
2.  **Paste the Final Final Code:**

```python
import evdev, pygame, time

# 1. Setup Audio Engine
pygame.mixer.init()
s0 = pygame.mixer.Sound('/usr/share/bucklespring/wav/ff-0.wav')
s1 = pygame.mixer.Sound('/usr/share/bucklespring/wav/ff-1.wav')

# 2. Connect to Trackpad (event8)
try:
    d = evdev.InputDevice('/dev/input/event8')
except Exception as err:
    print(f"Error: {err}. Check permissions.")
    exit()

# 3. Timer Variable
touch_start_time = 0

print("Smart Mechanical Mouse Driver: FINAL ACTIVATED 🚀")

for e in d.read_loop():
    # --- SMART TAP (Code 330) ---
    if e.type == 1 and e.code == 330:
        if e.value == 1: # Finger Down
            touch_start_time = time.time()
        elif e.value == 0: # Finger Up
            duration = time.time() - touch_start_time
            if duration < 0.15:
                # MODERATE TAP: Crisp 15% volume
                s0.set_volume(0.15) 
                s1.set_volume(0.12)
                s0.play(); s1.play()

    # --- PHYSICAL FORCE CLICK (Code 272) ---
    elif e.type == 1 and e.code == 272:
        if e.value == 1: # Press
            # HIGH IMPACT: Heavy 40% volume
            s0.set_volume(0.40) 
            s0.play()
        elif e.value == 0: # Release
            s1.set_volume(0.30)
            s1.play()
```

---

### Phase 3: The Ignition (`~/clicks_on.sh`)
This script handles the startup sequence. We added a **10-second sleep** to ensure the Wayland desktop is fully ready before the sounds start.

1.  **Open the file:** `nano ~/clicks_on.sh`
2.  **Paste the Final Final Code:**

```bash
#!/bin/bash
# Wait for the desktop session to settle
sleep 10

# Kill any existing processes
pkill buckle
pkill -f mouse_driver.py

# Start Global Keyboard (event23) at high gain
buckle -s /dev/input/event23 -g 100 &

# Start the Smart Mouse Driver
python3 /home/void/mouse_driver.py &

echo "Mechanical Mode: FULLY ACTIVATED 🚀"
```
3.  **Make it executable:** `chmod +x ~/clicks_on.sh`

---

### Phase 4: The Auto-Start Entry
This tells GNOME to run your script every time you log in.

1.  **Ensure the folder exists:** `mkdir -p ~/.config/autostart`
2.  **Create the entry:** `nano ~/.config/autostart/mechanical.desktop`
3.  **Paste this configuration:**
    ```ini
    [Desktop Entry]
    Type=Application
    Name=Mechanical Sounds
    Exec=/home/void/clicks_on.sh
    Hidden=false
    NoDisplay=false
    X-GNOME-Autostart-enabled=true
    Comment=Starts Bucklespring and Smart Mouse Driver
    ```

---

### Phase 5: The "Mute" Switch (`~/clicks_off.sh`)
For when you need silence in a library or meeting.

1.  **Create the file:** `nano ~/clicks_off.sh`
2.  **Paste this:**
    ```bash
    #!/bin/bash
    pkill buckle
    pkill -f mouse_driver.py
    echo "Silence mode: ACTIVATED 🤫"
    ```
3.  **Make it executable:** `chmod +x ~/clicks_off.sh`

---

### 📋 Final Profile Summary
| Component | Device Node | Volume/Gain | Logic |
| :--- | :--- | :--- | :--- |
| **Keyboard** | `event23` | **100** (High) | Direct Bucklespring |
| **Smart Tap** | `event8` | **0.15** (Moderate) | < 150ms touch |
| **Force Click** | `event8` | **0.40** (Heavy) | Hardware Button 272 |
| **Movement** | `event8` | **Silent** | > 150ms touch |



**You are all set!** Your MacBook now has a completely custom, time-filtered, volume-balanced mechanical feedback system. 

Since you've achieved this "final final" setup, would you like me to help you find a way to add a **visual indicator** (like a small icon in the top bar) to show when "Mechanical Mode" is active?
