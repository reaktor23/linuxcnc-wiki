# Setup of LinuxCNC on Debian 10 Buster + Mesa 7i96 + Mesa 7i85

## Helpful resources

- https://gnipsel.com/linuxcnc/uspace 
- https://github.com/jethornton/7i96 
- https://www.qtpyvcp.com/install/quick_start.html
- https://github.com/LinuxCNC/mesaflash
- https://forum.linuxcnc.org/27-driver-boards/35820-7i96-7i85s
- http://freeby.mesanet.com/7i96_7i85.zip
- http://freeby.mesanet.com/7i96_7i85s.zip

## Debian

- Download Debian 10 Buster [debian-10.4.0-amd64-netinst.iso](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)
- Make sure to select right image for your system architecture (amd64 in my case)
- Write image to USB stick, install on PC

## RT Kernel

First of all, update and upgrade your packages with `sudo apt update` followed
by a `sudo apt upgrade`. This might require a reboot afterwards.

Now search for latest RT Kernel with `apt search linux-image-4.19`

You'll get a list of kernels, look for those with `rt` in the name and select
the latest, `linux-image-4.19.0-9-rt-amd64/stable,now 4.19.118-2 amd64
Linux 4.19 for 64-bit PCs, PREEMPT_RT (signed)` in my case.

Then install this kernel with `sudo apt install linux-image-4.19.0-9-rt-amd64`
and reboot the machine.

The kernel should be the default automatically, at least it was in my case.

Verify that your kernel is the correct one by running `uname -a`, the result
should be `Linux mill 4.19.0-9-rt-amd64 #1 SMP PREEMPT RT Debian 4.19.118-2
(2020-04-29) x86_64 GNU/Linux`

## IP configuration

The Mesa 7i96 has 3 methods of IP configuration, selectable by the W5 jumpers
on the card.
By default, both jumpers are in the down position which means a fixed IP `192.168.1.121`.
We're not going to change that and adjust our PCs network setting accordingly.

To do so, go to `Applications -> Settings -> Advanced Network Configurtation`.
There you can select the interface that is connected to your Mesa card and set
the IP under IPv4 Setting to `192.168.1.1` and the netmask to `24`.
You can for sure select another IP, as long as it is in the same range as the
IP of the Mesa card.

Verify that your settings are correct with a ping to the Mesa card with
`ping 192.168.1.121` (it must be powered to do this ;-) )
The result should look like this:

```txt
PING 192.168.1.121 (192.168.1.121) 56(84) bytes of data.
64 bytes from 192.168.1.121: icmp_seq=1 ttl=64 time=0.086 ms
64 bytes from 192.168.1.121: icmp_seq=2 ttl=64 time=0.141 ms
64 bytes from 192.168.1.121: icmp_seq=3 ttl=64 time=0.117 ms
64 bytes from 192.168.1.121: icmp_seq=4 ttl=64 time=0.139 ms
^C
--- 192.168.1.121 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 60ms
rtt min/avg/max/mdev = 0.086/0.120/0.141/0.025 ms
```

## Install LinuxCNC

Follow these commands to install linuxcnc. Some of the packages are maybe not 
neccessary, but do no harm either.

```bash
sudo apt install devscripts build-essential git-core git-gui make gcc python-pip dh-python 
libudev-dev python-yapps tcl8.6-dev tk8.6-dev libreadline-gplv2-dev asciidoc dblatex docbook-xsl 
dvipng graphviz groff imagemagick inkscape python-lxml source-highlight texlive-extra-utils
texlive-font-utils texlive-fonts-recommended texlive-lang-cyrillic texlive-lang-french texlive-lang-german 
texlive-lang-polish texlive-lang-spanish texlive-latex-recommended w3c-linkchecker xsltproc asciidoc-dblatex 
python-dev libxmu-dev libglu1-mesa-dev libgl1-mesa-dev libgtk2.0-dev intltool libboost-python-dev netcat 
libmodbus-dev yapps2 gdebi python-tk libusb-1.0-0-dev libtirpc-dev

cd ~
git clone https://github.com/LinuxCNC/linuxcnc.git build
cd build/linuxcnc
```

At this point we need to modify the source in order to make it possible to build it.
Edit the `debian/control.bottom.in` file with nano or vim and search and remove these lines (the linenumbers might differ)

```txt
 14     python-gtksourceview2,
 15     python-vte | gir1.2-vte-2.91,
```

Now run `debian/configure uspace`, this should return `successfully configured for 'uspace-Debian-10'-'uspace'..` 
After that, run `dpkg-checkbuilddeps` which should not return any output. If it does, install the packages that are required by `dpkg-checkbuilddeps`.

Now build go into the debian folder with `cd debian` and build the linuxcnc deb with `debuild -uc -us`

This takes about 5 to 10 minutes, be patient :-)

Next, go to the build folder with `cd ..` and install linuxcnc with `sudo dpkg -i linuxcnc-uspace_2.9.0~pre0_amd64.deb`.

In my case that resulted in many missing dependencies, but I was able to install them with `sudo apt --fix-broken install`.

Now linuxcnc should be installed and able to start. to do so simply use the `linuxcnc` command.

## 7i96 configuration tool

Install the dependencies with `sudo apt install python3-pip python3-pyqt5 libpci-dev git`

Now install the tool itself with `pip3 install git+https://github.com/jethornton/7i96.git` and create a symlink with `sudo ln -s ~/.local/bin/7i96 /usr/bin/7i96`.

The tool can be started with the command `7i96`

**!! [](https://github.com/jethornton/7i96/issues/9) !!**

```bash
cp ~/.local/lib/python3.7/site-packages/m7i96/mesaflash64 ~/.local/bin
ln -s ~/.local/bin/mesaflash64 /usr/bin/mesaflash64`
```

## Install mesaflash

**!! This is not needed as mesaflash comes with the 7i96 tool !!**

Install dependencies with `sudo apt install libpci-dev pkg-config` and go to you home directory with `cd ~`.

Now clone the git repo with `git clone https://github.com/LinuxCNC/mesaflash.git`, afterwards cd into the new folder with `cd mesaflash`

Finally build and install mesaflash with `sudo make install`
