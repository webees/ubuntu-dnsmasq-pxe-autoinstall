# Install dnsmasq packages
```
apt install -y dnsmasq
systemctl stop systemd-resolved.service
systemctl disable systemd-resolved.service
systemctl enable --now dnsmasq.service
systemctl restart dnsmasq
```

# Configuring the dnsmasq service
```
tee > /etc/dnsmasq.conf << 'EOF'
server=1.1.1.1

bind-interfaces

dhcp-range=192.168.0.1,192.168.0.250,24h
dhcp-option=option:router,192.168.0.254
dhcp-option=option:ntp-server,192.168.0.254
dhcp-option=option:dns-server,192.168.0.254
dhcp-option=option:netmask,255.255.255.0

enable-tftp
tftp-root=/home/dnsmasq/tftp

dhcp-boot=/bios/pxelinux.0,pxeserver,192.168.0.253
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,grub/bootx64.efi

log-facility=/home/dnsmasq/dnsmasq.log
EOF
```

# Create the TFTP Folder Structure
```
mkdir -p /home/dnsmasq/tftp/{boot/live-server,bios,grub}

tftp
├── bios
│   ├── boot => /home/dnsmasq/tftp/boot
│   ├── ldlinux.c32
│   ├── libutil.c32
│   ├── lpxelinux.0
│   ├── menu.c32
│   ├── pxelinux.0
│   └── vesamenu.c32
├── boot
│   └── live-server
│       ├── initrd
│       └── vmlinuz
├── grub
│   ├── bootx64.efi
│   ├── font.pf2
│   └── grub.cfg
└── grubx64.efi
```

# Download UEFI Packages
```
apt-get download shim.signed
apt-get download grub-efi-amd64-signed

dpkg -x shim-signed*.deb shim
dpkg -x grub-efi*.deb grub

cp grub/usr/lib/grub/x86_64-efi-signed/grubnetx64.efi.signed /home/dnsmasq/tftp/grubx64.efi
cp shim/usr/lib/shim/shimx64.efi.signed                      /home/dnsmasq/tftp/grub/bootx64.efi
```

# Download pxelinux Packages
```
wget https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.zip
unzip syslinux-6.03.zip -d syslinux

cp syslinux/bios/com32/elflink/ldlinux/ldlinux.c32  /home/dnsmasq/tftp/bios
cp syslinux/bios/com32/libutil/libutil.c32          /home/dnsmasq/tftp/bios
cp syslinux/bios/com32/menu/menu.c32                /home/dnsmasq/tftp/bios
cp syslinux/bios/com32/menu/vesamenu.c32            /home/dnsmasq/tftp/bios
cp syslinux/bios/core/pxelinux.0                    /home/dnsmasq/tftp/bios
cp syslinux/bios/core/lpxelinux.0                   /home/dnsmasq/tftp/bios

ln -s /home/dnsmasq/tftp/boot /home/dnsmasq/tftp/bios/boot
```

# pxelinux
```
mkdir /home/dnsmasq/tftp/bios/pxelinux.cfg

tee > /home/dnsmasq/bios/pxelinux.cfg/default << 'EOF'
DEFAULT menu.c32
NU TITLE ULTIMATE PXE SERVER - By Griffon - Ver 2.0
PROMPT 0 
TIMEOUT 0

MENU COLOR TABMSG 37;40 #ffffffff #00000000
MENU COLOR TITLE 37;40 #ffffffff #00000000 
MENU COLOR SEL 7 #ffffffff #00000000
MENU COLOR UNSEL 37;40 #ffffffff #00000000
MENU COLOR BORDER 37;40 #ffffffff #00000000

LABEL Ubuntu Server 20.04.1
kernel /boot/live-server/vmlinuz
initrd /boot/live-server/initrd
append ip=dhcp ds=nocloud-net;s=http://192.168.0.253/autoinstall/user-data url=http://192.168.0.253/iso/ubuntu-20.04.5-live-server-amd64.iso
EOF
```
