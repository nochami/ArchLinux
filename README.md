#!/bin/bash

IFS=$'\n'
delim="------------------------------------"

echo -e "Welcome to Arch Linux installer!\n"

echo $delim

read -p "Enter the computer name: " comp_name
read -p "Create a new password for root: " comp_pass

echo $delim

echo -e "List of the file systems: \n1. ext2 \n2. ext3 \n3. ext4"
read -p "Select number " FC_num
i=0
while [ $i == 0 ]
do
	case $FC_num in
		1)
			FC=ext2
			break;;
		2)
			FC=ext3
			break;;
		3)
			FC=ext4
			break;;
		*)
			read -p "Uncorrect! Please, enter just numbers: " FC_num ;;
	esac
done

echo $delim

echo -e "List of the sections: \n1. sda1 \n2. sda2 \n3. sda3"
read -p "Select number " CASE_SECTION_NUM
while [ $i == 0 ]
do
	case $CASE_SECTION_NUM in
		1)
			SECTION=sda1
			SECTION_NUM=1
			break;;
		2)
			SECTION=sda2
			SECTION_NUM=2
			break;;
		3)
			SECTION=sda3
			SECTION_NUM=3
			break;;
		*)
			read -p "Uncorrect! Please, enter just numbers: " CASE_SECTION_NUM ;;
	esac
done

echo $delim

echo "Create a new user"
read -p "Enter username " username
read -p "Enter password " pass

echo $delim

echo -e "List of display managers: \n1. SDDM \n2. LXDM \n3. XDM"
read -p "Select number " DISPLAY_MANAGER

while [ $i == 0 ]
do
	if [[ !$DISPLAY_MANAGER =~ ^[1-3]+$ ]] ; 
	then 
		echo "Please, enter just numbers"
		read -p "Selest number " DISPLAY_MANAGER
	else
		break;
	fi
done

echo $delim

echo -e "List of dekstop environments : \n1. GNOME \n2. LXDE \n3. Xfce"
read -p "Select number " DEKSTOP_ENV

while [ $i == 0 ]
do
	if [[ !$DEKSTOP_ENV =~ ^[1-3]+$ ]] ;
	then
		echo "Please, enter just numbers"
		read -p "Select number " DEKSTOP_ENV
	else
		break;
	fi
done

echo $delim 

echo "Installation was beginning"

#echo -e "n\np\n1\n\n\nw\q" | fdisk /dev/sda

path="/dev/sda"
sed -e 's/\s*\([\+0-9a-zA-Z]*\).-/\1/' << EOF | fdisk $path
n
p
$SECTION_NUM
2048

w
e
EOF

#echo $(fdisk -l)
mkfs.$FC /dev/$SECTION
mount /dev/$SECTION /mnt
pacstrap /mnt base linux linux-firmware base-devel
genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt /bin/bash -c "ln -sf /usr/share/zoneinfo/Europe/Tallinn /etc/localtime"
arch-chroot /mnt /bin/bash -c "hwclock --systohc"
arch-chroot /mnt /bin/bash -c "yes | pacman -S vim nano networkmanager grub"
arch-chroot /mnt /bin/bash -c "systemctl enable NetworkManager"
arch-chroot /mnt /bin/bash -c "sed -i s/'#en_US.UTF-8'/'en_US.UTF-8'/g /etc/locale.gen"
arch-chroot /mnt /bin/bash -c "locale-gen"
arch-chroot /mnt /bin/bash -c "echo '$comp_name' > /etc/hostname"
arch-chroot /mnt /bin/bash -c "mkinitcpio -P"
echo "root:$comp_pass" | arch-chroot /mnt chpasswd
arch-chroot /mnt /bin/bash -c "grub-install /dev/sda"
arch-chroot /mnt /bin/bash -c "grub-mkconfig -o /boot/grub/grub.cfg"

arch-chroot /mnt /bin/bash -c "useradd -m $username -g root -p $(openssl passwd -crypt $pass) -s /bin/bash"

arch-chroot /mnt /bin/bash -c "pacman -Syu"
arch-chroot /mnt /bin/bash -c "pacman -S xorg xorg-xinit mesa --noconfirm"
case $DISPLAY_MANAGER in
	1)
		arch-chroot /mnt /bin/bash -c "pacman -S sddm-kcm --noconfirm"
		arch-chroot /mnt /bin/bash -c "systemctl enable sddm";;
	2)
		arch-chroot /mnt /bin/bash -c "pacman -S lxdm --noconfirm"
		arch-chroot /mnt /bin/bash -c "systemctl enable lxdm";;
	3)
		arch-chroot /mnt /bin/bash -c "pacman -S xorg-xdm --noconfirm"
		arch-chroot /mnt /bin/bash -c "systemctl enable xdm";;
esac

case $DEKSTOP_ENV in
	1)
		arch-chroot /mnt /bin/bash -c "pacman -S gnome gnome-extra --noconfirm";;
	2)
		arch-chroot /mnt /bin/bash -c "pacman -S lxde-common lxsession openbox --noconfirm";;
	3)
		arch-chroot /mnt /bin/bash -c "pacman -S xfce4 --noconfirm";;
esac
reboot
