Проверяю версию:
~# uname -r
6.8.0-55-generic
~# uname -p
x86_64

    3  mkdir kernel && cd kernel
    4  wget https://kernel.ubuntu.com/mainline/v6.13.7/amd64/linux-headers-6.13.7-061307-generic_6.13.7-061307.202503131244_amd64.deb
    5  wget https://kernel.ubuntu.com/mainline/v6.13.7/amd64/linux-headers-6.13.7-061307_6.13.7-061307.202503131244_all.deb
    6  wget https://kernel.ubuntu.com/mainline/v6.13.7/amd64/linux-image-unsigned-6.13.7-061307-generic_6.13.7-061307.202503131244_amd64.deb
    7  wget https://kernel.ubuntu.com/mainline/v6.13.7/amd64/linux-modules-6.13.7-061307-generic_6.13.7-061307.202503131244_amd64.deb
    8  dpkg -i *.deb
    9  ls -la /boot
   10  update-grub
   11  grub-set-default 0
   12  reboot

Снова проверяю версию:
uname -r
6.13.7-061307-generic
