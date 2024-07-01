<details><summary>My Fedora Config</summary>
  
```
# Remove uneeded applications
sudo flatpak uninstall -y --delete-data org.gnome.NautilusPreviewer org.gnome.Loupe org.gnome.TextEditor org.gnome.Evince org.gnome.Snapshot org.gnome.Connections org.gnome.font-viewer org.gnome.clocks org.gnome.Weather org.gnome.baobab org.gnome.Screenshot org.gnome.Maps org.fedoraproject.MediaWriter org.gnome.Calculator org.gnome.Calendar org.gnome.Characters org.gnome.Contacts org.gnome.Logs org.gnome.Extensions

# Configure flatpak repos
sudo flatpak remote-delete flathub
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Install NoiseTorch
wget https://raw.githubusercontent.com/mylesotoole/Install-NoiseTorch/main/Install-NoiseTorch && chmod +x Install-NoiseTorch && ./Install-NoiseTorch


# Configure Steam to start minimized at login
mkdir -p ~/.config/autostart
tee ~/.config/autostart/steam.desktop >/dev/null <<EOF
[Desktop Entry]
Type=Application
Name=Steam
Exec=flatpak run com.valvesoftware.Steam -silent
StartupNotify=false
Terminal=false
EOF

# Configure Discord to start minimized at login
tee ~/.config/autostart/discord.desktop >/dev/null <<EOF
[Desktop Entry]
Type=Application
Name=Discord
Exec=flatpak run com.discordapp.Discord --start-minimized
StartupNotify=false
Terminal=false
EOF

# Reset GNOME Configuration
dconf reset -f /

# Extensions
extensions=(
    "appindicatorsupport@rgcjonas.gmail.com"
)
for extension in "${extensions[@]}"; do
    # Check if the extension is already installed
    if ! gnome-extensions list | grep -q "$extension"; then
        echo "Installing extension: $extension"
        gdbus call --session \
                   --dest org.gnome.Shell.Extensions \
                   --object-path /org/gnome/Shell/Extensions \
                   --method org.gnome.Shell.Extensions.InstallRemoteExtension \
                   "$extension"
    else
        echo "Extension $extension is already installed"
    fi
done
gsettings set org.gnome.shell enabled-extensions "['appindicatorsupport@rgcjonas.gmail.com']"

# Window Management Settings
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
gsettings set org.gnome.desktop.wm.preferences auto-raise true
gsettings set org.gnome.desktop.wm.preferences focus-new-windows 'smart'
gsettings set org.gnome.desktop.wm.preferences num-workspaces 1
gsettings set org.gnome.mutter dynamic-workspaces false
gsettings set org.gnome.mutter center-new-windows true

# Power and Sleep Settings
gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 3600
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-timeout 3600

# Interface and Appearance Settings
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
gsettings set org.gnome.desktop.interface clock-show-date false
gsettings set org.gnome.desktop.interface clock-show-weekday true
gsettings set org.gnome.desktop.background picture-uri-dark file://'/usr/share/backgrounds/gnome/blobs-d.svg'
gsettings set org.gnome.desktop.background picture-uri file://'/usr/share/backgrounds/gnome/blobs-l.svg'

# Privacy and User Data Management Settings
gsettings set org.gnome.desktop.privacy remember-recent-files false
gsettings set org.gnome.desktop.privacy remove-old-trash-files true
gsettings set org.gnome.desktop.privacy remove-old-temp-files true
gsettings set org.gnome.desktop.privacy remember-app-usage false
gsettings set org.gnome.desktop.privacy hide-identity true
gsettings set org.gnome.desktop.privacy old-files-age 7
gsettings set org.gnome.system.location enabled false

# Notifications and Extensions Settings
gsettings set org.gnome.desktop.notifications show-in-lock-screen false

# Miscellaneous Settings
gsettings set org.gnome.desktop.input-sources xkb-options "['caps:none']"

# Install Flatpaks
flatpak -y install --user flathub org.gnome.Totem
flatpak -y install --user flathub org.gnome.Loupe
flatpak -y install --user flathub com.valvesoftware.Steam
flatpak -y install --user flathub org.gnome.Boxes
flatpak -y install --user flathub io.github.shiftey.Desktop
flatpak -y install --user flathub net.davidotek.pupgui2
flatpak -y install --user flathub com.heroicgameslauncher.hgl
flatpak -y install --user flathub com.discordapp.Discord
flatpak -y install --user flathub com.spotify.Client

# Configure Flatpak permissions
flatpak override --user --filesystem=home org.mozilla.firefox
flatpak override --user --filesystem=home com.discordapp.Discord
flatpak override --user --filesystem=/mnt com.valvesoftware.Steam

# Install Betterfox for Firefox
curl -O https://raw.githubusercontent.com/mylesotoole/Install-Betterfox/main/Linux && chmod +x Linux && ./Linux

# Default to X11
if grep -q "#WaylandEnable=false" /etc/gdm/custom.conf; then
    sudo sed -i 's/#WaylandEnable=false/WaylandEnable=false/' /etc/gdm/custom.conf
fi

# Configure layered packages and drivers
sudo rpm-ostree override remove yelp gnome-tour --install akmod-nvidia
sudo rpm-ostree kargs --append=rd.driver.blacklist=nouveau --append=modprobe.blacklist=nouveau --append=nvidia-drm.modeset=1 --append=initcall_blacklist=simpledrm_platform_driver_init

# Reboot
reboot
```
</details>
