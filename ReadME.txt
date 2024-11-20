ШАГ 1 Установка Войда, установка и запуск hyprland, dhcpcd, chrony, pipewire

sudo xbps-install chrony dhcpcd

sudo ln -s /etc/sv/dhcpcd /var/service
sudo sv up dhcpcd

sudo ln -s /etc/sv/chronyd /var/service
sudo sv up chronyd

mkdir .local
mkdir .local/pkgs
cd .local/pkgs

sudo xbps-install git foot noto-fonts-ttf dbus seatd polkit elogind mesa-dri

git clone https://github.com/void-linux/void-packages.git
git clone https://github.com/Makrennel/hyprland-void.git

cd void-packages
./xbps-src binary-bootstrap

cd..
cd hyprland-void
cat common/shlibs >> ~/.local/pkgs/void-packages/common/shlibs
cp -r srcpkgs/* ~/.local/pkgs/void-packages/srcpkgs
cd
cd ~/.local/pkgs/void-packages

./xbps-src pkg hyprland
./xbps-src pkg xdg-desktop-portal-hyprland
./xbps-src pkg hyprland-protocols

sudo xbps-install -R hostdir/binpkgs hyprland
sudo xbps-install -R hostdir/binpkgs hyprland-protocols
sudo xbps-install -R hostdir/binpkgs xdg-desktop-portal-hyprland

sudo ln -s /etc/sv/dbus /var/service
sudo ln -s /etc/sv/polkitd /var/service
sudo ln -s /etc/sv/seatd /var/service

sudo usermod -aG _seatd $USER

sudo xbps-install -S pipewire pipewire-devel wireplumber libpulseaudio pulseaudio-utils alsa-pipewire


sudo mkdir -p /etc/alsa/conf.d
sudo ln -s /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d
sudo ln -s /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d

ШАГ 2 Установка софта:

sudo xbps-install xorg-server-xwayland telegram-desktop firefox nemo tumbler foot grim slurp swayimg mpv mypdf
 Waybar wofi nerd-fonts-symbols-ttf fish-shell file-roller blender gimp inkscape ghostscript imlib2-webp
 webp-pixbuf-loader musescore pavucontrol obs obs-plugin-browser-bin cmus cmus-pulseaudio micro swaybg jq
 openjdk21 transmission-gtk

sudo xbps-install void-repo-multilib
sudo xbps-install -Suy
sudo xbps-install lutris MangoHud MangoHud-32bit wine winetricks wine-32bit mesa-dri-32bit
libGL-32bit libtxc_dxtn-32bit giflib giflib-32bit libpng libpng-32bit libldap libldap-32bit gnutls
gnutls-32bit libopenal libopenal-32bit v4l-utils v4l-utils-32bit libgpg-error libgpg-error-32bit
libjpeg-turbo libjpeg-turbo-32bit sqlite sqlite-32bit libXcomposite libXcomposite-32bit
libXinerama libXinerama-32bit libgcrypt libgcrypt-32bit ncurses ncurses-libs ncurses-libs-32bit
ocl-icd ocl-icd-32bit libxslt libxslt-32bit libva libva-32bit gst-plugins-base1
gst-plugins-base1-32bit amdvlk amdvlk-32bit vulkan-loader vulkan-loader-32bit libX11-devel
libX11-devel-32bit libgpg-error libgpg-error-32bit gdk-pixbuf gdk-pixbuf-32bit libgcc
libgcc-32bit libglvnd libglvnd-32bit mesa-vulkan-radeon mesa-vulkan-radeon-32bit psmisc
fluidsynth libunwind Vulkan-Tools Vulkan-Headers

sudo xbps-install apparmor

В /etc/default/grub добавить

CONFIG_SECURITY_APPARMOR=y
CONFIG_AUDIT=y
CONFIG_LSM="landlock,lockdown,yama,integrity,apparmor,bpf"
GRUB_CMDLINE_LINUX_DEFAULT="apparmor=1 security=apparmor"

sudo grub-mkconfig -o /boot/grub/grub.cfg

ШАГ 3 УСТАНОВКА ТЕМ, ИКОНОК И ОТКЛЮЧЕНИЕ КНОПОК

gsettings set org.gnome.desktop.interface icon-theme Flat-Remix-Black-Dark
gsettings set org.gnome.desktop.interface gtk-theme Flat-Remix-GTK-Grey-Dark
gsettings set org.gnome.desktop.interface cursor-theme capitaine-cursors-light
gsettings set org.gnome.desktop.interface font-name 'JetBrainsMono 10'
gsettings set org.gnome.desktop.wm.preferences button-layout :

ШАГ 4 УПРАВЛЕНИЕ ПИТАНИЕМ И АВТОЛОГИН

В visudo нужно добавить:
%wheel ALL=(ALL:ALL) NOPASSWD: /bin/shutdown

Для автологина /etc/sv/agetty-tty1/conf нужно привести к виду:

if [ -x /sbin/agetty -o -x /bin/agetty ]; then
	# util-linux specific settings
	if [ "${tty}" = "tty1" ]; then
		GETTY_ARGS="-a USERNAME --noclear"
	fi
fi

Для автостарта в ./config/fish/config.fish нужно добавить:

if status is-login
    if test -z "$DISPLAY" -a "$(tty)" = /dev/tty1
        dbus-run-session Hyprland
    end
end
