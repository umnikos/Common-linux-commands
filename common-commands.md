
# Useful googling flags
  - `“[query]”` - exact match
  - `site:[url]` - limit search results to that site
  - `-[term]` - remove results including that term
  - `before:[year]`, `after:[year]` - limit search results by issue date
     - useful for getting up-to-date news
     - `[year]..[year]` - this but for a year range
  - `([terms] | [terms]) [more terms]` - search twice with each version and then merge the results
  - `*` - a wildcard (use anywhere)
  - `filetype:[type]` - limit results to that filetype
     - useful for filtering out scam links when searching for a pdf

# Do stuff with pipes in fish shell
## merge two pipes into one sequentially
  - with blocks: 
    ```fish
    begin
       echo 1
       echo 2
    end
    ```
  - with `echo`:
    ```fish
    echo (cmd1) (cmd2)
    ```
  - with `cat`:
    ```fish
    cat (cmd1 | psub) (cmd2 | psub)
    ```

## give two pipes to a command simultaneously
  ```fish
  cmd (cmd1 | psub) (cmd2 | psub)
  ```

## split one pipe into two (copying the data)
   - not possible yet
      - https://github.com/fish-shell/fish-shell/issues/1786

# Do stuff with ffmpeg
## webm to mp3
  - don't, instead convert to ogg to avoid re-encoding:
    `ffmpeg -i "input.webm" -vn -c:a copy "output.ogg"`

## Combining audio and video files into one
  - `ffmpeg -i video.webm -i audio.webm -c copy -map 0:v:0 -map 1:a:0 output.webm`
     - `-i video.webm -i audio.webm` - input both files
     - `-c:v copy -c:a copy` - do not re-encode, just copy the streams
        - `-c copy` does the same thing
     - `-map 0:v:0 -map 1:a:0` - pick video from the first file and audio from the second

## Concatenate two videos
  - create `list.txt` with all of your files to concatenate:```txt
     file one.webm
     file two.webm```
  - then run the following command:
     - `ffmpeg -f concat -safe 0 -i list.txt -c copy output.webm`

## Convert 1fps video to 30fps video (for recording presentations)
  - TODO

# Unzipping and zipping different zip formats
## .sqsh / .squashfs
  - mount: `mount -o loop file.sqsh your/dir`
     - `-o loop` to create a block device backed up by the squash file
  - unzip: `unsquashfs file.sqsh`

## .tar.gz
  - unzip: `tar xzf name.tar.gz`
  - zip: `tar czf name.tar.gz [files]`

## .tar.xz
  - unzip: `tar xJf name.tar.xz`

## .gz
  - zip: `gzip file.txt`
  - unzip: `gunzip file.gz`

## .iso
  - mount: `mount -o loop file.iso your/dir`
     - `-o loop` to create a block device backed up by the iso file
  - zip: `mkisofs -o out.iso dir`

## .rar
  - unzip: `unrar e name.rar`

## .zip
  - unzip: `unzip name.zip`
  - zip: `zip name.zip ./*`

## .7z
  - unzip: `7z e name.7z`

# Bindiff two files
  - In fish:
     - `diff (xxd one | psub) (xxd two | psub)`
  - In bash:
     - `diff <(xxd one) <(xxd two)`

# Check if two files are identical (even when they’re on different machines)
  - `md5sum file.txt`
  - `md5sum otherfile.txt`
  - compare the sums

# Finding a file in a directory with find
  - searching by name
     - `find . -name “filename.txt”`
  - searching for recently edited files
     - `find . -mmin 1`
  - search only in the current directory
     - `find . -maxdepth 1 -name “filename.txt”`
  - negate a pattern
     - `find . -type f -not -name “filename.txt”

# Find package that contains a command you want to install
  - https://command-not-found.com/

# TXT file re-encoding (for subtitles)
  - Recognizing encoding:
     - `enca -L bg ./*`
  - Converting to utf8:
     - `enconv -L bg -x utf8 ./*`
  - Note: `bg` is the language here, `enca/enconv` doesn’t seem to support that many

# Mass-renaming of files
  - `rename s/from/to ./*`

# Getting rid of HPLIP’s error on startup
  - edit `/usr/share/hplip/ui5/systemtray.py` and comment out the responsible code: 
  ```python
      if not QSystemTrayIcon.isSystemTrayAvailable():
      # FailureUI(None,
      #     QApplication.translate("SystemTray",
      #     "<b>No system tray detected on this system.</b><p>Unable to start, exiting.</p>",
      #     None),
      #     QApplication.translate("SystemTray", "HPLIP Status Service",
      #     None))
   ```

# Setting monitor brightness below the minimum
  - `sudo su -c ”echo $argv[1] > /sys/class/backlight/intel_backlight/brightness”`

# Make graph in terminal
  - With spark:
     - `spark 100 50 10 40 60 110`

# Get what each open port is being used by
  - `sudo ss -ltp`
     - `-l` - listening
     - `-t` - tcp
     - `-p` - show processes
     - `sudo` - because you can’t see daemons
  - alternatively: `sudo netstat -ltp`
     - if `netstat` is not installed: `apt install net-tools`

# SSH port forwarding / SSH tunneling
  - local: client:rebound-port → server → destination-name:destination-port
     - `ssh -N user@server  -L rebound-port:destination-name:destination-port`
  - remote: server:rebound-port → client → destination-name:destination-port
     - `ssh -N user@server  -R rebound-port:destination-name:destination-port`
  - dynamic: same as local but done to every client port
     - `ssh -N user@server  -C -D local-port-for-socks-proxy`
        - `-C` means “Compression”, is optional
  - `-N` is optional and tells ssh to not launch a shell (“No command”)

# VPN through SSH
  - `sshuttle -r user@host 0/0`
     - `0/0` is shorthand for `0.0.0.0/0` and means “any ip address”
     - optionally add `--dns` to resolve dns queries through the vpn

# Generate RSA keypair for SSH
  - `ssh-keygen`
  - generate on client, move public key to server
  - move public key with `ssh-copy-id -i ~/.ssh/mykey.pub user@host`

# Configure SSH server to not allow password authentication / Disable SSH password auth
  - edit /etc/ssh/sshd_config:
     - `PasswordAuthentication no`
     - `UsePAM no`
     - `PermitRootLogin no`
     - `PermitRootLogin prohibit-password`
  - then `ssh systemctl restart ssh`

# Only allow SSH connections from localhost (useful in combination with ssh tunneling)
  - edit /etc/ssh/sshd_config:
     - `ListenAddress ::1`
     - `ListenAddress 127.0.0.1`
  - then `ssh systemctl restart ssh`

# Several hostnames for the same SSH host
  - add them to /etc/hosts instead of /.ssh/config

# Reverse `sshfs`
  - reconsider
     - there is no easy way
     - better setup an ftp server
  - install ssh on the client as well and connect to it from the server
     - ssh tunnel if firewalls are a problem

# Change default java version
  - `sudo update-alternatives --config java`
  - if your version of java isn’t listed you can add it with:
  - `sudo update-alternatives --install /usr/bin/java “java” /path/to/bin/java 1`
     - the structure is `--install softlink-location option-name binary-path priority`
     - example with java from snap: `sudo update-alternatives —install /usr/bin/java ”java” /snap/openjdk/current/jdk/bin/java 1`

# How to go back to gdm3 (gnome) after getting stranded in i3/xmonad/whatever
  - `sudo dpkg-reconfigure gdm3`

# Extracting Outlook .msg files
  - sources
     - https://superuser.com/questions/99250/opening-a-msg-file-in-ubuntu
     - https://softwarerecs.stackexchange.com/questions/13470/extract-attachments-from-eml-file
  - `sudo apt install libemail-outlook-message-perl libemail-sender-perl mpack`
  - `msgconvert file.msg`
  - `munpack file.eml`

# Lock pc with a terminal command
  - `xdg-screensaver lock`

# Play video in vlc from youtube link with youtube-dl
  - `youtube-dl “https://www.youtube.com/watch?v=vELps9Qu6e8” -o - | vlc -`
     - for audio only you can run `youtube-dl` with `-F` to see the available file formats and then `-f 249` (or whatever number) to choose format with only audio

# Impress someone with pretty command-line things
  - visually pleasing
     - `cbonsai`
     - `curl wttr.in`
     - `bpytop`
     - `cmatrix`
     - `xeyes`
  - not so visually pleasing
     - `toilet`
     - `fortune`
     - `cowsay`
     - `yes`

# Setting up sudoers file (/etc/sudoers)
  - basic setup:
  ```bash
  # User privilege specification
  root	ALL=(ALL:ALL) ALL

  # Allow members of group sudo to execute any command
    %sudo	ALL=(ALL:ALL) ALL
  ```
  - if you want to be able to run a particular command without typing out your password:
  ```bash
  %sudo ALL=(ALL:ALL) ALL, NOPASSWD: /usr/bin/apt
  ```

# Optimize linux for an SSD
  - https://goinglinux.com/articles/Optimize_Linux_For_SSD_Drive_en.htm
## Move tmp files to RAM
  - add the following to `/etc/fstab`: 
  ```bash
  tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0
  tmpfs /var/log tmpfs defaults,noatime,mode=0755 0 0
  tmpfs /var/spool tmpfs defaults,noatime,mode=1777 0 0
  tmpfs /var/tmp tmpfs defaults,noatime,mode=1777 0 0
  ```

## TODO - the rest

# Printing from the command line (after you’ve setup your printer)
  - `lp file.pdf`
  - before printing, check the default options are the correct ones:
     - `lpoptions`
  - if they’re not, edit them temporarily or permanently with `-o`:
     - `lp/lpoptions  -o sides=one-sided  file.pdf`
        - check the man pages for all available options

# Allow flatpak to read your entire home directory
  - `flatpak override --user --filesystem=home:ro com.discordapp.Discord`
     - remove `:ro` to make it read-write

# Fix firefox flatpak funky/aliased fonts (very obvious on xckd.com)
  - add the following to ~/.var/app/org.mozilla.firefox/config/fontconfig/fonts.conf: 
  ```xml
      <?xml version='1.0'?>
      <!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
      <fontconfig>
          <!-- Disable bitmap fonts. -->
          <selectfont><rejectfont><pattern>
              <patelt name="scalable"><bool>false</bool></patelt>
          </pattern></rejectfont></selectfont>
      </fontconfig>
  ```

# Change linux user’s password
  - `passwd`
     - `sudo passwd myusername` to change a particular user’s password

# Get basic system info
  - easy way: `neofetch`
  - hard way: `uname`

# Different types of files in linux (and their appearance in `exa -l`)
  - `.` - regular file
  - `d` - directory
  - `|` - pipe
  - `l` - soft link
  - `b` - block device (drive/partition)
  - `c` - character device (serial)
  - `s` - socket files

# OpenWrt WiFi keeps disconnecting and is slow to connect in the first place
    Network → Wireless → [Your WiFi] → Edit → Interface Configuration → Advanced → Disassociate On Low Acknowledgement → Set to unchecked
                                                                             ... → Short Preamble → Set to unchecked
                                                                             ... → DTIM → Set to 3
                                                                  ... → General Setup → WMM Mode → Set to checked
                                        ... → Device Configuration → Advanced → Country Code → [Your current country]
                                                                          ... → Beacon Interval → Set to 100
  - if ghosts still haunt you, delete all settings and start over
  - also, check if your WAN cable isn’t getting unplugged
  - for what these options mean:
     - https://openwrt.org/docs/guide-user/network/wifi/basic

# Running a QEMU VM
  - `qemu-system-x86_64`, and then a bunch of flags:
     - `-cdrom image.iso` - what iso image to boot from (when installing or livebooting)
     - `-m 2G` - allocate 2GB of ram for the vm
     - `-boot order=d` - boot order: cdrom drive, then whatever. optional as that’s the default
     - `-enable-kvm` - enable kvm acceleration
     - `-drive` - mount a drive, a bunch of options follow that as well
        - `file=image.iso,media=cdrom` - mount an iso image as a drive
        - `file=hard_disk_file_name,format=qcow2` - mount a qcow2 hdd file
           ▹ created with: `qemu-img create -f qcow2 hard_disk_file_name 16G`
        - `file=/dev/sdc,format=raw` - mount a real drive as a drive
           ▹ requires sudo
           ▹ cannot mount just a partition btw

# Clear cache for a single web page in firefox
  - Ctrl+F5
     - F5 is just the refresh button, this refreshes and clears the cache

# Testing adblocking
  - see if it blocks ads
     - https://ads.malev.ru/
     - https://d3ward.github.io/toolz/adblock.html
  - see if adblock detectors see your adblocker
     - https://www.detectadblock.com/

# Simulating a circuit (before you go and pay for parts, build it and then realize it doesn’t work)
  - https://www.falstad.com/circuit/

# Copying large amounts of data locally or remotely (e.g. to make backups)
  - `zpaq`
  - `rsync`
  - `rdiff-backup`

# Setting up torrenting
  - use a daemon and a remote client rather than a standalone app
  - port forward your router
     - why?: https://bt.degreez.net/firewalled.html
  - set your encryption mode to “require encryption”
     - it costs you nothing and may prevent your shitty ISP from messing with your torrenting

# List of wargames/ctfs
  - big list: https://github.com/apsdehal/awesome-ctf
  - small list: https://gist.github.com/fakhrullah/e8794f4847f3114316235ad7b0530dec

# Add custom search engine to firefox
  - https://mycroftproject.com
     - you can submit a custom url if the site you want to add doesn’t have a plugin yet
     - based on: https://superuser.com/a/1566328/953870
     - they seem to be stored in ~/.mozilla/firefox/[profile]/search.json.[random] but I can’t figure out the encoding syntax

# Play movie in the terminal with vlc
  - `env DISPLAY=”” vlc file.mp4`

# Linux users management
## Creating a new linux user
  - `adduser [username]`
     - friendlier than `useradd`

## Deleting a linux user
  - `deluser [username]`

## Adding a linux user to a group
  - `usermod -a -G [group] [user]`

# Remind me why crypto/web3/blockchain sucks
  - https://www.stephendiehl.com/blog/web3-bullshit.html
  - https://moxie.org/2022/01/07/web3-first-impressions.html

# I hate it so much when I copy something in my browser and it’s not what I actually copied because of fucking javascript
  - for firefox: go to `about:config`
     - set `dom.event.clipboardevents.enabled` to false
  - while you’re doing that, might as well protect yourself from look-alike links based on unicode
     - set `network.IDN_show_punycode` to true

# How to manage Haskell packages
  - do not use `cabal`. ever.
     - cabal hell is real
     - cabal will make you uninstall everything ever over and over again
  - maybe use `stack`?

# How to manage Python packages
  - do not use `pip`. ever.
     - different pip for each version of python
        - and you’re gonna have many versions of python
  - use `python3 -m pip` instead of `pip`

# Changing default permissions files and directories are created with
  - temporarily: `umask [num]`
     - `num` is to be subtracted from `777` for dirs and `666` for files
  - permanently:
     - futile:
        - edit /etc/profile and add `umask [num]`
        - create /etc/profile.d/umask.sh with just `umask [num]` in it
        - edit /etc/login.defs with the new umask
        - edit /etc/pam.d/login and add `session required pam_umask.so umask=[num]` to it
        - edit /dev/null and add `[your cries of defeat]` to it
     - ugly and doesn’t work with gnome
        - edit ~/.bashrc and add `umask [num]`

# Installing grub
  - https://wiki.manjaro.org/index.php/GRUB/Restore_the_GRUB_Bootloader

# Booting from grub rescue
  - `ls` to see drives and partitions
  - `ls (hd0,msdos1)` to see partition’s file system
  - `ls (hd0,msdos1)/` to see what’s inside (the `/` means root, you can change it to `/boot` or anything you want)
  - find which partition has `/boot` in it, I'll assume it's `(hd0,msdos1)`
  - `set boot=(hd0,msdos1)`
  - `set prefix=(hd0,msdos1)/boot/grub`
  - `insmod normal` (install mod)
  - `normal` (run mod)

# Program for drawing category theory diagrams
  - https://tikzcd.yichuanshen.de/

# Http-only site?
  - httpforever.com

# Finding a fast apt mirror repository
  - `netselect-apt`

# Editing gnome icons
  - located at /usr/share/applications/
  - edit the right `.desktop` file
  - done (no need to reload)

# How to make Steam not eat your CPU while idling
  - https://www.reddit.com/r/Steam/comments/f3o1lw/do_you_hate_the_new_steam_library_heres_how_to/

# Isopropanol (rubbing alcohol) recipes
  - shoe stretch: 33%
  - disinfectant: 70%?
  - glasses cleaner: 25% (15%-70%???)

# Restart X server
  ```bash
  sudo systemctl restart display-manager
  pulseaudio -k
  ```

# Getting more osu maps
  - beatmap packs
     - osu dans are a thing (under “theme”)
  - multiplayer lobbies

# Set a ram limit on an application
  - `systemd-run --scope --user -p MemoryLimit=2G google-chrome`

# Designing a motherfucking website
  - https://motherfuckingwebsite.com/
  - http://bettermotherfuckingwebsite.com/
  - https://evenbettermotherfucking.website/
  - https://thebestmotherfuckingwebsite.co/

# Unmount sshfs/fuse filesystem
  - `fusermount -u some/dir`

# Install newer debian package using backports
  - `apt install <package>/bullseye-backports`

# How does thunar eject without sudo?
  - `udisksctl mount/umount/power-off -b /dev/sdc1`

# Convert pdf to excel table
  - `pdftotext -layout`
  - `tabula`

# Apt is pretending the gpg signature I just added doesn’t exist!
  - `chmod o+r` the gpg file
     - yeah. it’s that stupid

# Virtmanager Windows VM optimizations
  - https://leduccc.medium.com/improving-the-performance-of-a-windows-10-guest-on-qemu-a5b3f54d9cf5
  - https://wiki.archlinux.org/title/Intel_GVT-g
     - if you have two GPUs then you don't need to virtualize a GPU and can just directly passthrough the second one
  - looking glass??? (I couldn't get it to work myself)

# Send a file from place to place
  - `wormhole` (magic wormhole)
  - https://send-anywhere.com/
  - torrenting
  - syncthing
  - kde connect
  - ssh
     - `sftp`
     - `sshfs`

# Fucking locales messing up my ssh session
  - `sudo locale-gen ”en_US.UTF-8”` (probably not necessary)
  - `export LC_ALL=C`
  - `sudo dpkg-reconfigure locales`
  - if no errors were generated then reboot

# Printing things in order
 ```fish
  for f in (find . -type f -maxdepth 1 -exec ls -tr1 {} +)
      lp -o media=a4 $f
  end 
  ```


