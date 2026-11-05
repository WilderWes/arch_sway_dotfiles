# Arch Sway Dotfiles

Author: Weston Preising
Date: 2026-09-26

## Intro

An opinionated Arch sway (Wayland) setup for development.
High level, this is a fairly basic sway setup which emphasizes vim-based workflows and applications.
Ported from my [arch_i3wm_dotfiles](https://github.com/WilderWes/arch_i3wm_dotfiles); `sway/config` is the upstream default sway config (1.12) with my i3 keybinds merged in.
Some audio/video tools included for recording guitar.
Arch (`core`, `extra`, `multilib`) + [CachyOS repos](https://wiki.cachyos.org/features/optimize_repos/) on btrfs with `linux-cachyos`; AUR via `paru`.

## Pitfalls

- sway does **not** run XDG autostart `.desktop` files; everything is started with `exec` in `sway/config`.
- Session variables in `/etc/environment`:
  - `QT_QPA_PLATFORMTHEME=qt6ct`
  - `WLR_RENDERER=vulkan` -- required for sway to read colors from EDID
  - `_JAVA_AWT_WM_NONREPARENTING=1` -- fixes blank java windows, see [wiki](https://wiki.archlinux.org/title/Sway#Java_applications)
- Color: sway 1.12 `color_profile --device-primaries gamma22` (reads EDID) + `render_bit_depth 10`
- Fractional scaling: `eDP-1 scale 1.66667`; XWayland apps (anything X11-only) render blurry

## Arch Setup

- [linux-cachyos](https://wiki.cachyos.org/features/kernel/) -- desktop kernel; `linux-zen` and `linux` as fallbacks
- [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot) -- bootloader (UEFI, ESP mounted at `/boot`)
  - `systemd-boot-manager` -- pacman hook regenerates entries on kernel install/removal; manually: `sudo sdboot-manage gen` (`update-boot` in fish)
  - kernel cmdline lives in `LINUX_OPTIONS` in `/etc/sdboot-manage.conf`, then regenerate
- [zram-generator](https://wiki.archlinux.org/title/Zram) -- swap in RAM; `zswap.enabled=0`
- [paru](https://aur.archlinux.org/packages/paru) -- AUR helper
- [cloudflare-warp](https://developers.cloudflare.com/warp-client/get-started/linux/) -- more secure internet, verify via [dnsleaktest.com](https://www.dnsleaktest.com/)
  - [systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved#DNS) -- stops warp/NetworkManager racing over `/etc/resolv.conf`
    - `sudo ln -sf ../run/systemd/resolve/stub-resolv.conf /etc/resolv.conf && sudo systemctl enable --now systemd-resolved`
- [downgrade](https://aur.archlinux.org/packages/downgrade) -- makes it easy to rollback a broken pkg
- [pacman-contrib](https://wiki.archlinux.org/title/Pacman) -- `pactree` shows dependency tree
- [reflector](https://wiki.archlinux.org/title/Reflector) -- `--country US --protocol https --latest 10 --sort rate` in `/etc/xdg/reflector/reflector.conf`, then `sudo systemctl enable --now reflector.timer`
- [alsa-firmware alsa-tools alsa-utils sof-firmware](https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture) -- fixes thinkpad mute/micmute leds
- [ly](https://github.com/fairyglade/ly) -- tui-based display manager (`sudo systemctl enable ly@tty1`)
  - [gnome-keyring seahorse](https://wiki.archlinux.org/title/GNOME/Keyring) -- keyring for 1password; `seahorse` is the keyring GUI
- [cups cups-pk-helper system-config-printer](https://wiki.archlinux.org/title/CUPS) -- printer support (`sudo systemctl enable --now cups.socket`)
  - [avahi nss-mdns](https://wiki.archlinux.org/title/Avahi#Hostname_resolution) -- network printer discovery (`sudo systemctl enable --now avahi-daemon`)
    - `/etc/nsswitch.conf`: `hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns`
    - `MulticastDNS=resolve` in `/etc/systemd/resolved.conf`, see [wiki](https://wiki.archlinux.org/title/Systemd-resolved#mDNS)
    - `sudo ufw allow 5353/udp`
- [gammastep](https://gitlab.com/chinstrap/gammastep) -- bluelight filter (redshift fork with wayland support); `gtk3 libappindicator-gtk3 python-gobject python-xdg` for `gammastep-indicator` tray icon
- [fonts](https://wiki.archlinux.org/title/Fonts) -- `noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-jetbrains-mono`
- [fontconfig](https://wiki.archlinux.org/title/Font_configuration) -- font rendering (HiDPI is `output eDP-1 scale` in `sway/config`)
- security
  - [ufw](https://wiki.archlinux.org/title/Uncomplicated_Firewall) -- uncomplicated firewall (`sudo systemctl enable --now ufw`)
  - [1password](https://1password.com/) -- password manager desktop app (`$mod+p`)
- [qt5ct](https://github.com/desktop-app/qt5ct) + [qt6ct](https://github.com/trialuser02/qt6ct) + [adw-gtk-theme](https://github.com/lassekongo83/adw-gtk3) -- system wide darkmode, see [reddit thread](<https://www.reddit.com/r/hyprland/comments/1h4abmt/how_do_i_apply_dark_theme/>)
  - `gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'`
  - `gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3-dark'`
  - `echo 'QT_QPA_PLATFORMTHEME=qt6ct' | sudo tee -a /etc/environment`
  - `echo '_JAVA_AWT_WM_NONREPARENTING=1' | sudo tee -a /etc/environment` -- fixes blank java windows, see [wiki](https://wiki.archlinux.org/title/Sway#Java_applications)
  - native wayland for qt6 apps ships in `qt6-base`; `qt5-wayland` only matters if a qt5 app other than `qt5ct` gets installed
- [fwupd udisks2 udisks2-btrfs](https://wiki.archlinux.org/title/Fwupd) -- firmware updates via `fwupdmgr`
- [intel graphics](https://wiki.archlinux.org/title/Hardware_video_acceleration) -- `mesa intel-media-driver vpl-gpu-rt vulkan-intel lib32-vulkan-intel lib32-mesa intel-compute-runtime`, see [sway wiki](https://github.com/swaywm/sway/wiki)
  - `intel-gpu-tools` -- GPU monitoring & troubleshooting (`sudo intel_gpu_top`)
  - `libva-intel-driver` -- legacy i965 VA-API driver, only for pre-Broadwell GPUs; unused on Arc/Lunar Lake (`xe`), which uses `intel-media-driver`
- [pipewire](https://wiki.archlinux.org/title/PipeWire) -- `pipewire-alsa pipewire-pulse pipewire-jack pavucontrol`
  - wireplumber -- disable audio suspension; prevents clipping when audio stops/starts
- [gstreamer](https://wiki.archlinux.org/title/GStreamer) -- `gst-libav gst-plugin-pipewire gst-plugin-va gst-plugins-base gst-plugins-good`
- [docker](https://wiki.archlinux.org/title/Docker) -- `sudo systemctl enable --now docker && sudo usermod -aG docker $USER`
  - [devcontainer-cli](https://github.com/devcontainers/cli) -- `npm install -g @devcontainers/cli` (lands in `~/.local/bin` via `.npmrc`)
- [localsend](https://localsend.org/) -- cross-platform file sharing, see [ufw profile](https://github.com/localsend/localsend/issues/1230)

## Current Languages

- git
- bash/zsh/fish
- [latex](https://wiki.archlinux.org/title/TeX_Live) -- `texlive biber`, vimtex + zathura
- C/C++ -- `clang`
- python -- `uv`
  - `python-virtualenv python-matplotlib python-networkx python-pandas python-scipy python-scikit-learn python-pyarrow python-tabulate ipython`
  - `python-jupytext` (AUR) -- notebooks as plain text
- java -- `jdk25-openjdk`
- go
- javascript/typescript -- `npm`
- rust -- `rust rust-analyzer`

## Documentation

- [zeal](https://github.com/zealdocs/zeal) -- store offline documentation for various languages
- [anki](https://apps.ankiweb.net/) -- flashcards

## Sway Setup

- [sway](https://wiki.archlinux.org/title/Sway) -- `sway swaybg swayidle swaylock xorg-xwayland`
  - `ly` picks up `/usr/share/wayland-sessions/sway.desktop` automatically
- [xdg-desktop-portal-wlr xdg-desktop-portal-gtk](https://wiki.archlinux.org/title/XDG_Desktop_Portal) -- screensharing + file picker
- input (touchpad, `ctrl:nocaps`, flat accel) -- `input` blocks in `sway/config`, see `man 5 sway-input`
  - caps lock → ctrl, flat pointer accel, tap-to-click, natural scrolling, disable-while-typing
- [kanshi](https://wiki.archlinux.org/title/Kanshi) -- auto switch laptop/docked (DP-1 or HDMI-A-1) outputs; `swaymsg -t get_outputs` for names
- [lxqt-policykit](https://github.com/lxqt/lxqt-policykit) -- program to manage app privileges
- [network-manager-applet](https://wiki.archlinux.org/title/NetworkManager) -- show network icon in swaybar (`nm-applet --indicator`)
- [bluez bluez-utils blueman](https://wiki.archlinux.org/title/Bluetooth) -- bluetooth stack + GUI (`sudo systemctl enable --now bluetooth`)
- [waybar](https://github.com/Alexays/Waybar) -- status bar (`swaybar_command waybar` in `sway/config`)
  - `otf-font-awesome` -- icons used in `waybar/style.css`
- [power-profiles-daemon](https://wiki.archlinux.org/title/CPU_frequency_scaling#power-profiles-daemon)
- [rofi](https://github.com/davatorium/rofi) -- app launcher
- [xdg-user-dirs xdg-utils](https://wiki.archlinux.org/title/XDG_user_directories) -- standard home dirs (`xdg-user-dirs-update`) and `xdg-open`
- [wl-clipboard](https://github.com/bugaevc/wl-clipboard) -- clipboard (`wl-copy`/`wl-paste`)
- [swayidle](https://github.com/swaywm/swayidle) + [swaylock](https://github.com/swaywm/swaylock) -- lock after 5 min, suspend after 6 min, `$mod+Escape` to lock
- [grim](https://sr.ht/~emersion/grim/) + [slurp](https://github.com/emersion/slurp) -- screenshot area to clipboard (`Print`)
- [brightnessctl](https://github.com/Hummer12007/brightnessctl) -- enable brightness control keys
- [playerctl](https://github.com/altdesktop/playerctl) -- media keys
- Note: [set battery thresholds](https://discussion.fedoraproject.org/t/how-to-set-battery-health-threshold/175092)
  `echo 80 | sudo tee /sys/class/power_supply/BAT0/charge_control_end_threshold`
  `echo 75 | sudo tee /sys/class/power_supply/BAT0/charge_control_start_threshold`

## Terminal & Shell

- [kitty](https://github.com/kovidgoyal/kitty) -- JetBrains Mono NL and Tokyo Night theme
  - [ghostty](https://ghostty.org/download) -- JetBrains Mono NL and Tokyo Night theme
- [fish](https://fishshell.com/)
- [starship](https://starship.rs/) -- terminal prompt
- [eza](https://github.com/eza-community/eza) -- ls replacement
- [fastfetch](https://github.com/fastfetch-cli/fastfetch) -- fetch system info on shell start
- [cpdf](https://github.com/coherentgraphics/cpdf-binaries) -- PDF manipulation CLI
- [man-db man-pages](https://wiki.archlinux.org/title/Man_page) -- local man pages
- [tldr](https://github.com/tldr-pages/tldr) -- quick cli tips n' tricks
- [cheat.sh](https://github.com/chubin/cheat.sh) -- additional cli tips n' tricks
  - `curl https://cht.sh/:cht.sh > ~/.local/bin/cht.sh && chmod +x ~/.local/bin/cht.sh && cht.sh --standalone-install`
  - `--standalone-install` also needs `git python-virtualenv`
- [bear](https://github.com/rizsotto/Bear) -- reads compile commands from make and adds hints to `clangd`
- [ripgrep-all](https://github.com/phiresky/ripgrep-all) -- ripgrep but for pdfs, docx, epub, etc.
  - `pandoc-bin` -- docx/epub/odt adapter; `poppler` (pdf) and `ffmpeg` (media) come from the yazi line
- [lazygit](https://github.com/jesseduffield/lazygit) -- git TUI
- [htop](https://htop.dev/) + [btop](https://github.com/aristocratos/btop) -- process viewers

## Editor & File Management

- [yazi](https://yazi-rs.github.io/docs/installation#arch) -- terminal file manager
  - `sudo pacman -S yazi ffmpeg 7zip jq poppler fd ripgrep fzf zoxide resvg imagemagick`
- [thunar thunar-archive-plugin gvfs](https://wiki.archlinux.org/title/Thunar) -- GUI file manager with trash, auto-mount, extract/compress
- [neovim](https://neovim.io/) -- configured via [LazyVim](https://www.lazyvim.org/)
  - [luarocks](https://luarocks.org/) -- lua package manager, lazy.nvim uses it for plugins with rockspecs
- [vscodium-bin](https://vscodium.com/) -- as needed for school when turning in jupyter notebook assignments
- [zathura zathura-pdf-poppler](https://pwmt.org/projects/zathura-pdf-poppler/) -- for PDF viewing (dark mode, clipboard selection)
- [libreoffice-fresh](https://wiki.archlinux.org/title/LibreOffice)
  - see fonts, hunspell, etc on wiki
- [mpv](https://mpv.io/) + [imv](https://sr.ht/~exec64/imv/) -- video and image viewers
- [dropbox](https://wiki.archlinux.org/title/Dropbox) -- cloud file sync

## Browsers

- [firefox](https://www.firefox.com/en-US/) -- main browser
  - [DarkReader](https://darkreader.org/)
  - [uBlock Origin](https://ublockorigin.com/)
  - [SponsorBlock](https://sponsor.ajay.app/)
  - [Unhook](https://unhook.app/)
  - [1Password](https://1password.com/downloads/browser-extension)
  - [bypass-paywalls](https://gitflic.ru/user/magnolia1234)
- [qutebrowser](https://qutebrowser.org/) used strictly for [Markdown Preview](https://github.com/iamcco/markdown-preview.nvim)
- [chromium](https://wiki.archlinux.org/title/Chromium) -- used for chrome specific needs (i.e. Google Meet 1080p support)

## Communication

- [vesktop](https://vesktop.dev/) -- a better discord
- [zoom](https://zoom.us/)

## Music Production

- [realtime-privileges](https://wiki.archlinux.org/title/Realtime_process_management) -- `sudo usermod -aG realtime $USER`
- [obs-studio](https://obsproject.com/) -- recording/streaming
  - via flatpak since [droidcam] better supported there
  - `v4l2loopback-dkms usbmuxd` -- virtual camera + droidcam over usb
- [transcribe](https://www.seventhstring.com/) -- slow down audio, loop sections, transcribe by ear
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) -- download videos/audio from YouTube
- [bitwig-studio](https://www.bitwig.com/) -- DAW
- [davinci-resolve-studio](https://www.blackmagicdesign.com/products/davinciresolve) -- video editing

## AI Usage

- Using minimally to find better primary source materials or as a socratic tutor for debugging/optimizing code
- used to audit this install and update this README 2026-10-05
