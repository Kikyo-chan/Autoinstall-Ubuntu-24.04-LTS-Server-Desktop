# Automate the installation of Ubuntu24.04 LTS Live-Server and Ubuntu24.04 LTS Desktop using Cloud-Init and Subiquity technologies. 

## Abstract
This document describes the steps to automate the installation of the Ubuntu 22.04.x LTS Desktop using Cloud-Init and Subiquity technologies.   


## Introduction
The Ubuntu24.04 LTS Live-Server and Ubuntu24.04 LTS Desktop ISO uses Subiquity, which means that the
Subiquity Autoinstall format is used for automation.
* [Desktop Latest images](https://cdimage.ubuntu.com/daily-live/current/).
* [Server Latest images](https://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/).

For more information on Autoinstall, please see:
* [Automated Server Installs](https://ubuntu.com/server/docs/install/autoinstall)
* [Automated Server Install Quickstart](https://ubuntu.com/server/docs/install/autoinstall-quickstart)
* [Automated Server Installs Config File Reference](https://ubuntu.com/server/docs/install/autoinstall-reference)

## Ubuntu24.04 LTS Desktop Autoinstall (user-data)
```yaml
#cloud-config
# See the autoinstall documentation at:
# https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html
autoinstall:
  refresh-installer:
    update: true
    channel: latest/edge
  apt:
    disable_components: []
    fallback: abort
    geoip: false
    mirror-selection:
      primary:
      - country-mirror
      - arches: &id001
        - amd64
        - i386
        uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
      - arches: &id002
        - s390x
        - arm64
        - armhf
        - powerpc
        - ppc64el
        - riscv64
        uri: http://ports.ubuntu.com/ubuntu-ports
    preserve_sources_list: false
    security:
    - arches: *id001
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
    - arches: *id002
      uri: http://ports.ubuntu.com/ubuntu-ports
  codecs:
    install: true
  drivers:
    install: true
  kernel:
    package: linux-generic-hwe-24.04
  keyboard:
    layout: us
    variant: ''
  locale: en_US.UTF-8
  network:
    version: 2
    ethernets:
        ens33:
            match:
                name: "en*"
            dhcp4: true
        eth1:
            match:
                name: "eth*"
            dhcp4: true
  oem:
    install: auto
  source:
    id: ubuntu-desktop-minimal
    search_drivers: true
#  ssh:
#    allow-pw: true
#    authorized-keys: []
#    install-server: true
  storage:
    layout:
      name: lvm
      sizing-policy: all
  identity:
    realname: ''
    hostname: ubuntu-desktop
    username: ubuntu
    password: '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0'
  timezone: Asia/Shanghai
  updates: security
  late-commands:
#    - curtin in-target --target /target -- apt update -y
#    - curtin in-target --target /target -- apt upgrade -y
#    - curtin in-target --target /target -- apt-get dist-upgrade -y
#    - curtin in-target --target /target -- apt autoremove -y
#    - chmod -R 600 /target/etc/netplan/
    - curtin in-target --target=/target -- sudo usermod -p '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0' root
    - reboot
  version: 1
```
## Ubuntu24.04 LTS Server Autoinstall(user-data)
```yaml
#cloud-config
autoinstall:
  refresh-installer:
    update: true
    channel: latest/edge
  apt:
    disable_components: []
    fallback: abort
    geoip: false
    mirror-selection:
      primary:
      - country-mirror
      - arches: &id001
        - amd64
        - i386
        uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
      - arches: &id002
        - arm64
        uri: http://ports.ubuntu.com/ubuntu-ports
    preserve_sources_list: false
    security:
    - arches: *id001
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
    - arches: *id002
      uri: http://ports.ubuntu.com/ubuntu-ports
  packages:
    - git
    - curl
    - wget
    - vim 
    - build-essential
  drivers:
    install: true
  refresh-installer:
    channel: edge
    update: yes
  storage:
    layout:
      name: lvm
      sizing-policy: all
  user-data:
    disable_root: false
  identity:
    realname: ''
    hostname: ubuntu
    username: ubuntu
    # password is xxxxxxxxx
    password: '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0'
#  kernel:
#    package: linux-generic
  keyboard:
    layout: us
    toggle: null
    variant: ''
  locale: en_US.UTF-8
  network:
    version: 2
    ethernets:
        ens33:
            match:
                name: "en*"
            dhcp4: true
        eth1:
            match:
                name: "eth*"
            dhcp4: true
  oem:
    install: auto
  snaps:
  - channel: stable
    classic: true
    name: powershell
  source:
    id: ubuntu-server
    search_drivers: true
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
  updates: security
  timezone: Asia/Shanghai
  late-commands:
    - curtin in-target --target /target -- apt update -y
    - curtin in-target --target /target -- apt upgrade -y
    - curtin in-target --target /target -- apt-get dist-upgrade -y
    - curtin in-target --target /target -- apt autoremove -y
    - curtin in-target --target=/target -- /usr/bin/wget -P /etc/netplan/ http://172.17.80.238/ubuntu/netcfg/server-netcfg.yaml
    - chmod -R 600 /target/etc/netplan/
    - curtin in-target --target=/target -- sudo usermod -p '$6$jMIpAD1dGkm6sqfJ$HF6uWZp3lbYMjyI3jWUE1j51R9sqCkX8tjrp3xut2AWs3r2Ou9JGyZu1Xr7D3VF3B4X2gYCfjQatKoxAbDGSu0' root
    - curtin in-target --target=/target -- update-grub2
    - curtin in-target --target=/target -- echo "Please wait...will reboot automatically"
    - reboot
  version: 1
```
  ### PXE&iPXE BOOT Config
  1. Ubuntu24.04 LTS Desktop PXE&iPXE BOOT Config
     ```yaml
     set base-url http://172.17.100.221/iso/ubuntu/24.04_desktop/noble-desktop
     kernel ${base-url}/casper/vmlinuz
     initrd ${base-url}/casper/initrd
     set ubuntu_iso_url ${base-url}/noble-desktop-amd64.iso
     imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=8388608 cloud-config-url=/dev/null url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     #imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=8388608 url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     boot || goto failed
     ```
  2. Ubuntu24.04 LTS Server PXE&iPXE BOOT Config
     ```yaml
     set base-url http://172.17.100.221/iso/ubuntu/24.04_live_server
     kernel ${base-url}/casper/vmlinuz
     initrd ${base-url}/casper/initrd
     set ubuntu_iso_url ${base-url}/noble-live-server-amd64.iso
     #imgargs vmlinuz initrd=initrd ip=dhcp root=/dev/ram0 ramdisk_size=1688608 url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     imgargs vmlinuz initrd=initrd ip=dhcp cloud-config-url=/dev/null url=${ubuntu_iso_url} autoinstall ds=nocloud-net;s=${base-url}/
     boot || goto failed
     ```
 ### Other
  1. 24.04_live_server Preview of system files and directories at deployment：
      ![image](https://github.com/user-attachments/assets/53763127-7a58-4f45-9f17-fe9ef858e5b6)
     
  2. 24.04_live_noble-desktop Preview of system files and directories at deployment：
     ![image](https://github.com/user-attachments/assets/07e5a623-8418-42ea-a26b-a53be429da25)

  4. 5




