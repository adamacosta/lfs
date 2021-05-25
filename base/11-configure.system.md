# Configure the LFS system

## Network configuration

```sh
cat << EOF > /etc/systemd/network/10-ethernet.network
[Match]
Type=ether

[Network]
DHCP=yes
IPv6PrivacyExtensions=yes

[DHCPv4]
RouteMetric=512

[DHCPv6]
RouteMetric=512
EOF

ln -sfv /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## Configure hostname and hosts file

```sh
echo "lfs" > /etc/hostname
cat > /etc/hosts << EOF
# Begin /etc/hosts

127.0.0.1 localhost.localdomain localhost
::1       localhost ip6-localhost ip6-loopback
ff02::1   ip6-allnodes
ff02::2   ip6-allrouters

# End /etc/hosts
EOF
```

## Configure time

```sh
cat > /etc/adjtime << EOF
0.0 0 0.0
0
LOCAL
EOF
```

## Configure locale

```sh
cat > /etc/locale.conf << EOF
LANG=en_US.UTF-8
EOF
```

## Configure readline

```sh
cat > /etc/inputrc << EOF
# Begin /etc/inputrc

# do not bell on tab-completion
set bell-style none

set meta-flag on
set input-meta on
set convert-meta off
set output-meta on

# for linux console and RH/Debian xterm
"\e[1~": beginning-of-line
"\e[4~": end-of-line
"\e[5~": beginning-of-history
"\e[6~": end-of-history
"\e[7~": beginning-of-line
"\e[3~": delete-char
"\e[2~": quoted-insert
"\e[5C": forward-word
"\e[5D": backward-word
"\e\e[C": forward-word
"\e\e[D": backward-word
"\e[1;5C": forward-word
"\e[1;5D": backward-word

# for rxvt
"\e[8~": end-of-line

# for non RH/Debian xterm, can't hurt for RH/DEbian xterm
"\eOH": beginning-of-line
"\eOF": end-of-line

# for freebsd console
"\e[H": beginning-of-line
"\e[F": end-of-line

# End /etc/inputrc
EOF
```

## Configure shells

```sh
cat > /etc/shells << EOF
# Begin /etc/shells

/bin/sh
/bin/bash
/bin/zsh
/usr/bin/zsh

# End /etc/shells
EOF
```

Change default shell to `zsh`:

```sh
sed -i 's/\/bash/\/zsh' /etc/default/useradd
```

## /etc/os-release

Normally, `Linux From Scratch` does not include this file, but some packages will expect it to exist in order to get information about the installed system. Notably, if you decide to install the Glasgow Haskell Compiler, the recommended way to do it uses a tool `ghcup` that relies upon this file at least existing. It doesn't have to recognize the distro, but will throw an exception if the file does not exist at all. So we will create one:

```sh
cat > /usr/lib/os-release << EOF
NAME="Linux From Scratch"
PRETTY_NAME="Linux From Scratch"
ID=lfs
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://www.linuxfromscratch.org/"
DOCUMENTATION_URL="https://www.linuxfromscratch.org/"
SUPPORT_URL="https://www.linuxfromscratch.org/support.html"
BUG_REPORT_URL="https://www.linuxfromscratch.org/patches/submit.html"
LOGO=linuxfromscratch
EOF

ln -sfv ../usr/lib/os-release /etc/os-release
```

## Configure users

Ensure root cannot login and change default shell to `zsh`:

```sh
passwd -l root
chsh -s /bin/zsh
```

Create a non-root user and set a passwd:

```sh
useradd -m -G wheel -s /bin/zsh lfs
passwd lfs
```

Login as this user and you will go through `zsh` setup:

```sh
su lfs
```

Afterwards, set the prompt and colorize the `ls` command:

```sh
echo "autoload -U colors && colors" >> ~/.zshrc
echo "PS1=\"[%(#.%F{red}.%F{blue})%n%f@%F{blue}%m %1~%f]%(#.#.$) \"" >> ~/.zshrc
echo "eval $(dircolors)" >> ~/.zshrc
echo "alias ls='ls --color=auto'" >> ~/.zshrc
```

Copy the shell init file to `root`'s home:

```sh
sudo cp /home/lfs/.zshrc /root
```

Feel free to set this up how you like. This is simply what I prefer.

## Configure ssh keys

At this point, you'll want to add any public ssh keys you wish to use to login to your new user's `authorized_keys` file. If you have keys stored in GitHub, the easiest way to do this, from outside the chroot:

```sh
sudo su -
mkdir -p $LFS/home/lfs/.ssh
wget https://github.com/${USER}.keys -O $LFS/home/lfs/.ssh/authorized_keys
```

Then, from inside the chroot:

```sh
chown lfs:lfs /home/lfs/.ssh
chown lfs:lfs /home/lfs/.ssh/authorized_keys
chmod 750 /home/lfs/.ssh
chmod 640 /home/lfs/.ssh/authorized_keys
```